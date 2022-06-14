---
title: 我的博客新样式
time: '2022-06-15'
update: '2022-06-15'
description: '我的博客样式又又又更新啦!! _(:з'
tags: 'Web'
hidden: false
---

发现我对于写文章, 更喜欢折腾博客本身, 本末倒置了属于是.

### 起因

前阵子我在 macOS 上的类 Spotlight App 改用 [Raycast](https://www.raycast.com/) 了, 主要是它的扩展开发比起别的同类应用来说, 做到了开箱即用的集成环境, 这点十分的友好.

> 插播个小广告~有兴趣的童鞋还可以去看看我的个人开发并使用的 [Raycast 扩展合集](https://github.com/kayanouriko/raycast-extensions).

回到正题, 它的扩展开发使用到的是 [React](https://reactjs.org/) , 再写过几个小用例之后, 虽然写起来比 [Vue](https://vuejs.org/) 麻烦, 但是个人感觉写起来比起 Vue 更流畅舒服. 

可能是我 swiftUI 写多了, 这种嵌在 js 里面的写法感觉很舒服, 于是又萌生了再次改造博客的想法.

### 改造

#### React
接触前端也有一段时间了, 经过这次 React 重写之后, 感觉总算是入门了, 写出了比较满意的一个 Web 前端小项目了.

#### TypeScript
目前对于 [TypeScript](https://www.typescriptlang.org/) 的写法, 基本只停留在用它定义几个接口, 范型的简单应用, 其他写法和 js 基本没啥区别.

还没有遇到过 ts 进阶用法的业务场景, 在 [bilibili](https://www.bilibili.com/) 看 ts 的进阶教程也基本停留在理论阶段, 这玩意看时理解, 没遇到过使用场景加深印象的话, 基本上很快就忘记了.

> 现在才意识到笔记的重要性, 很多东西, 是我看过, 但是遇到问题又想不起写法, 手忙脚乱上网搜, 这时候笔记的作用就体现出来了.  

> 目前在 Github 建了一个笔记仓库, 平时的学习笔记基本一股脑儿的怼上去, 支持 VSCode Web, 还有全平台同步, 爽歪歪.

希望以后能在 ts 的理解能有更进一步的加强吧.

#### Tailwind CSS

之前一直用 [sass](https://sass-lang.com/) 来写样式的, 说是用 sass, 其实就是使用它的嵌套功能制造一些代码写法上的便利而已. 所以想了一下, 何不直接用原生的直接写呢. 去了解了一下, 认识到了 [Tailwind CSS](https://tailwindcss.com/).

网上对他的评价褒贬不一, 很多人看到类似这样的代码基本眉头紧皱.

```react
<div className="py-6 bg-white dark:bg-black flex justify-center shadow-sm">
    <!-- 更多布局标签 -->
    ...
</div>
```

刚开始学的时候, 基本每个样式都要翻文档, 简直是一种折磨. 但是熟悉之后, 就能发现, 其实常用就那么几个, 速度蹭蹭的往上提. 

而且个人项目拥抱现代特性, flex 一把梭的前提下, tailwindcss 确实香得不行, 我的博客项目完全没有建任何一个 css 文件. 顺带还轻松解决了响应式布局, 暗黑模式两大痛点.

要说缺点, 可能就是 class 夹在布局标签内, 当你的组件复杂度提升的时候, 会对你的代码逻辑理解造成一定程度上的干扰, 但是只要做好组件拆分规划, 这种程度的问题是在可以接受的范围内.

#### 其他

说实话, Web 前端的生态环境着实让人羡慕, 啥都有:

* `front-matter` 预处理了文章 json 文件
* `html-react-parser` 和 `showdown` 解决了 markdown 转 html 的问题
* `@tailwindcss/typography` 代替了之前一直使用的 github markdown 样式

一套组合拳下来, 加上 `Github Actions` 和 `Github Pages`. 一个可维护的, 自动化的博客并没有想象中的难写.

最后贴上一个我项目的依赖项:

```
"dependencies": {
    "@headlessui/react": "^1.6.4",
    "axios": "^0.27.2",
    "html-react-parser": "^1.4.14",
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "react-router-dom": "^6.3.0",
    "showdown": "^2.1.0"
},
"devDependencies": {
    "@tailwindcss/typography": "^0.5.2",
    "@types/node": "^17.0.40",
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "@types/showdown": "^2.0.0",
    "@vitejs/plugin-react": "^1.3.0",
    "autoprefixer": "^10.4.7",
    "front-matter": "^4.0.2",
    "postcss": "^8.4.14",
    "tailwindcss": "^3.0.24",
    "ts-node": "^10.8.1",
    "typescript": "^4.6.3",
    "vite": "^2.9.9"
}
```

#### 还在计划或者没加上的

本来这次改造想加上评论的, 已经考察好了使用的框架([giscus](https://github.com/giscus/giscus)), 但是想了一下自己的写作频率和访问人数, 还喜欢增删改, 实在没必要整个让自己尴尬的功能(bushi.

还有一些像 rss, 标签, 搜索, 文章内导航等功能, 一方面是功课还没做好, 另一方面同上理由需求量确实不大也都先搁置了. 不过以后肯定会出于练手的缘故也会加上的吧? 大..大概吧...

弄完这次改造, 想好好学习一下 WWDC22 先 _(:з

### 特别感谢

* Seseren 老师的 [b站空间](https://space.bilibili.com/305550122)
> 之前一直找不到好看带透明的加载图片, 直到我突然想起了 seseren 老师!!! 

> 感谢老师提供了博客里基本不会显示的高清透明的加载表情包图片.

* 头像和主页背景图均来自我最喜欢的动画[《这个美术社大有问题！》](https://zh.wikipedia.org/zh-cn/%E9%80%99%E5%80%8B%E7%BE%8E%E8%A1%93%E7%A4%BE%E5%A4%A7%E6%9C%89%E5%95%8F%E9%A1%8C%EF%BC%81)