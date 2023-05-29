---
title: "也许，你真的不需要 jQuery"
date: 2015-08-27 23:31:00 +0800
tags: [js]
---
<div style='text-align:center;'>
  <img src='/images/jqeury-no.png'/>
</div>

在 web 发展的历史过程中，jQuery 扮演了非常重要的角色，相对写原生的 javascript ，相同功能代码量少很多，语法简单，对新手友好。

这点大家可以通过 [you-dont-need-jquery](http://blog.garstasio.com/you-dont-need-jquery/) 系列文章对比两者代码差异。
该文章主要讲述的是如何使用原生 javascript 来实现 jQuery 主要功能，
目的是为了让大家深入理解 jQeury 背后的原理知识，知其所以然。

但作者肯定并不是真的让大家完全抛弃 jQuery ，直接裸写 js 的，想想，也没人这么做吧！

但时代总是发展变化的，而且很快，尤其是在移动设备占据了大部分的网络流量的情况下， jQuery 的地位和作用显得有些尴尬了。

对于传统网站，它还是第一选择，但是越来越多的 web 应用 (SPA) ，交互和功能太过复杂，它的表现就没那么好用了（相比很多现代的框架）。

背景： 最近在写 [mario-ui](https://github.com/giga-opensource/mario-ui), 采用的是 Flux 架构，依赖 [react](http://facebook.github.io/react/) 。

下面就从项目具体实践来谈谈，为什么此项目不需要 jQuery。

### 1. 选择器

react 有声明 ref 的功能， 例如在一个表单中有一个 `email` 的 ref, 我可以通过 `this.refs.email.getDOMNode()` 来达到同样效果。
```
<form className="login-block__form" onSubmit={this.onSubmit}>
  <input className="form__input login-block__input" placeholder="Email" type="email" ref='email'/> type="submit" />
</form>
```


### 2. DOM 操作

要知道 DOM 操作是一件性能很低下的操作，它会导致 DOM refresh/rerender.

由于 react 使用了 VIRTUAL DOM 的技术，DOM 操作这个根本就不存在，它直接根据 component 的 status，指定需要显示的内容。

### 3. Ajax 请求

我们使用 [SuperAgent](https://visionmedia.github.io/superagent/) 来处理 Ajax 请求，它只负责请求相关的问题，因为专注，所以做的更好，更多，更人性！

> Super Agent is light-weight progressive ajax API crafted for flexibility, readability, and a low learning curve after being frustrated with many of the existing request APIs. It also works with Node.js!

### 4. 事件绑定

我们使用 [events](https://github.com/Gozala/events) 来处理事件监听和触发。

### 5. 一些帮助方法

jQeury 提供了一些帮助方法，例如对象合并的 extend， 数据迭代操作！

我们项目使用了 [object-assign](https://github.com/sindresorhus/object-assign) 来处理对象合并。
数据迭代操作之类，可是使用[immutable](https://github.com/facebook/immutable-js),虽然现在项目还没开始使用，因为直接写的 for 循环！

### 6. animate 动画

对于现在浏览器而言，css animate 表现越来越好，这个也是未来趋势！个人觉得 animate 就属于 css 范畴。

### 7. 生态和插件

jQeury 这么多年了，插件是一抓一大把，react 虽然才出道 2 年多，但插件也不少，到 [npmjs](https://www.npmjs.com/search?q=react) 随便一搜，都有5000左右。 社区也越来越活跃，要知道前端造轮子的速度真的超乎想象。

项目目前为止用到的一些插件：

```
"react-dropzone": "^1.3.0",
"react-modal": "^0.3.0",
"react-paginate": "^0.1.35",
"react-router": "^0.13.3",
```

### 8. 浏览器兼容性

好吧，如果真的要谈这个，我只想说，I don't care, 毕竟现在浏览器更新很快，看看 IE 份额，直接 IE11 ，好么！

总结： 在很多 SPA 的项目中， jQeury 这种大而全的库也许不再适合，你完全可以根据自己需求， DIY 插件库。按需使用，总是不错的！

最后，推荐大家尝试下 react 以及一起来玩 [mario-ui](https://github.com/giga-opensource/mario-ui)。
