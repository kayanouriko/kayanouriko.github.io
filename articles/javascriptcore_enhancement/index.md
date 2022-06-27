---
title: 在 JavaScriptCore 中使用模块导入
time: '2022-06-27'
update: 
description: 学习如何在 JavaScriptCore 中注入全局函数和对象, 并模仿 nodejs 实现模块化功能
tags: swift, javascript
hidden: false
---

> 本文的 Demo 可以在 [jsc-require-swift
](https://github.com/kayanouriko/jsc-require-swift/) 获取

目前即使是原生开发, 也难免会和 Web 打交道. 借助 js 强大的生态库, 有时候是可以为原生开发提供更加便捷的业务解决方案. 现在也有很多应用借助 js 来服务或者增强功能. 例如 [JSBox](https://xteko.com/demos/en) 这样利用 js 去完全自定义原生界面, 或者 [bob](https://ripperhe.gitee.io/bob/#/) 利用 js 开发第三方翻译接口插件. 

Apple 为此在 iOS7 / macOS10.9 的时候就推出了 [JavaScriptCore](https://developer.apple.com/documentation/javascriptcore) 框架.

通常来说, `JavaScriptCore` 已经足够好用, 但是对于交互来说, 依旧有两个痛点 Apple并没有帮我们实现.

1. console.log 功能
2. module 导入功能

反观 js 的文档可以获知, console 功能其实是由环境平台实现的, 比如 Chrome, Firefox, VSCode等等, 每家实现的方式也不一样. 

同理我们需要打印调试信息到 Xcode 的 Console 也需要自行实现.

同样, js 需要支持模块导入, 涉及到文件 I/O 相关的接口, 这个同样需要 `Foundation` 框架 `File System` 的支持, 也需要我们自己实现.

所以下面我们正式开始着手解决这两个问题, 说实话我前期做功课找资料的时候, 中文方面基本还没有文章涉及到这两个问题的解决呢.

> 继续下面阅读之前, 你需要了解 `JavaScriptCore` 的基本使用, 这篇文章并不会详细的讲解 `JavaScriptCore` 的基础用法

### Console 的简易实现

原理其实比较简单, 就是在 `JSContext` 的运行环境中注入一个全局的 `console` 对象就好了.

在本文中, 为了还原浏览器中的 console 的基本使用, 定义了如下的代码.

```js
const console = {
    log: (...message) => {
        _consoleLog(message.join(" "))
    }
}
```
`JSContext` 中使用 `evaluateScript(_:)` 将上述 js 代码注入即可在环境内获取一个 `console` 对象.

其中 `_consoleLog` 为约定提供给 swift 调用的方法, 在此方法中, 我们可以获取 js 传递过来的参数.

> 此处 `message.join(" ")` 是由于尽管我们定义了剩余参数 message 为数组, 但是 swift 接收的时候会变的难以处理, 嵌套 any 过多, 所以提前在 js 中先转化, 这样 swift 使用 String 接收的话就能直接打印而不用做任何处理.

注入完成, 接下来我们需要实现 `_consoleLog` 方法, 调用 swift 的 `print` 将传入的 message 打印即可.

不幸的是, js 运行时要执行原生代码, 目前 `JavaScriptCore` 除去 c 的实现接口外, 只能定义 OC 的代码块, 所以不得不使用很恶心很难看的 `@convention(block)` 语法去封装 swift 闭包, 桥接成 oc 代码块.

```swift
// 定义 oc 代码块
let callback: @convention(block) (String?) -> Void = {
    print("[JavaScriptCore]: \($0 ?? "undefined")")
}
// 调用 setObject(_:forKeyedSubscript:) 桥接方法
jsContext.setObject(callback, forKeyedSubscript: "_consoleLog" as NSCopying & NSObjectProtocol)
```

至此, 我们可以在 js 文件内简单的使用 console 功能了, 并且使用习惯和浏览器中的基本保持一致.

当然在此处只是实现了最基础的功能, 结合 swift, 完全可以做到 js 原生一些进阶用法:
* debug 级别 info、error、warn等
* 输出 debug 到文件
* dir 打印对象信息
* time 和 timeEnd 计算程序执行消耗
* group 调试消息分组

### Module 的简易实现

#### nodejs 的模块实现原理

在 nodejs 模块化中, 使用 `exports` 或 `module.exports` 都能够向外部模块导出对象.

```js
const a = 3
const sayHello = (name) => {
    console.log('Hello, ' + name)
}

module.exports = {
    a,
    sayHello
}
```

而想要导入模块, 则需要用 `require` 方法.

通过查看 nodejs 的底层, 可以知道对于模块化的实现是借助了 `eval` 于 `fs` 模块来实现,将文件内容包裹在一个函数内反向导出.

诶, 这就巧了, `JavaScriptCore` 也有 `eval` 对应的方法 `evaluateScript(_:)`, 剩下的就简单了, 通过 `FileManager` 去查找导入的 js 文件即可.

#### 具体实现

首先我们约定一个 `require` 方法, 可以让 js 文件可以使用 commonjs 规范的写法导入模块.

```js
const utils = require("utils")
```

而在对应的 swift 实现代码如下:

```swift
let context = self
let callback: (String) -> JSValue = { jsName in
    // 1. 
    var moduleName = jsName
    if (jsName.hasSuffix(".js")) {
        moduleName = jsName.replacingOccurrences(of: ".js", with: "")
    }
    let path = Bundle.main.path(forResource: moduleName, ofType: "js") ?? ""
    let moduleContent = try? String(contentsOfFile: path, encoding: .utf8)
    // 2.
    let js = """
    (() => {
        let module = {}
        module.exports = {}
        let exports = module.exports
        \(moduleContent ?? "")
        return module.exports
    })()
    """
    return context.evaluateScript(js)
}
```

1. 这里只关注了在 target 里面的 js 导入情况, 在实际业务中, 一般来说这些 js 导入的操作应该是作为类似插件的效果, 存放在 document 等文件夹内. 此时应该用 FileManager 进行操作获取相关路径

2. 模仿 nodejs 的 require 实现原理. 简单来说, 就是将 js 文件内容包裹在一个函数内, 对外导出 module.exports. 借助 evaluateScript 功能, 很好实现.

由此可以看出, 模块导入的功能其实和 console 一样, 通过注入一个全局的方法, 获取到要导入的 js 内容, 注入到 JSContext 中即可.

### 总结

当然, 在项目中, 可能还需要一些细节逻辑需要优化, 但是整体来说, 实现打印和导入功能并不是很难.

另外其实我们也可以从 js 方面入手, 通过 babel 等工具其实也可以将多个模块合并, 这样就不需要 swift 方面介入了.

### 感谢

* [jsc-require](https://github.com/igorski89/jsc-require)
* [How to import modules in Swift's JavaScriptCore?](https://stackoverflow.com/questions/48354804/how-to-import-modules-in-swifts-javascriptcore)
* [浅析 commonjs 中的模块化实现原理](https://juejin.cn/post/6844904013310197767)




