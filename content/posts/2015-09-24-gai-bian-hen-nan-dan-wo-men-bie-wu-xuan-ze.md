---
title: "改变很难，但我们别无选择"
date: 2015-09-24 23:31:00 +0800
tags: [ruby]
---

MATTER (公司项目) 从严格意义上讲，应该是 SPA 的应用， 前端是 B/M(backbone + marionette) ＋ React,
后端是 Rails, 项目正全面往 React 迁移，不久将完全抛弃 B/M，目前属于中间状态，所有两种技术代码都有 。

该项目并未真正前后端分离，使用的还是 Rails 那套最标准的网站开发和部署流程，前端 JS 代码和应用一起打包发布。

很长一段时间里，Rails 自带的 [Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html) 工作的很好。
但随着开发进行，前端 JS 代码越来越多，每次代码修改，刷新浏览器，需要等待的时间太长。我想最主要原因还是 Rails 的 Asset Pipeline 慢，
而且大量代码是 coffeescript ，此情况更甚之。

终于有一天，我们再也无法忍受了， 决定来次大改变！

我们做了哪些改变：

1. 使用 webpack 取代 Asset Pipeline 来处理 JS 编译。
2. 使用 npm 安装 JS 插件，尤其是 React 相关。
3. 将所有 React 相关代码迁移到单独的目录，留给 webpack 使用。

改变过后的目录结构：

![giga_folder.png](/images/giga_folder.png)

这样的改变，给我们带来了很多好处：

1. JS 编译速度提升，页面还可以做到部分跟新，避免全页面刷新，从而保证开发速度。
2. 前端代码模块化问题自然解决。
3. 终于可以使用 npm 包发布的插件。

总结：  
使用 webpack 取代 Asset Pipeline 是一个正确的选择，前端代码日趋复杂，编译速度，模块化的支持越来越重要，所以我们才下定决定替换了它。
其实 Rails 就单单作为 api 接口也挺好，前端代码组织管理就交给 JS 吧。
