---
title: "如何在微信中进行优雅写作"
date: 2017-11-10 23:29:41
tags: [效率]
---

作为一名 Coder，如果浪费时间在微信排版之类的问题，显然是不划算的，今天我们一起来讨论下一些改进的方案。

### 缘起

半年来，断断续续写过十多遍文章，虽然数量不多，但我已感受到了微信自带网页编辑器的难用。

- 代码高亮支持极差，以至于我会贴图。
- 编写的内容，在网页、手机预览中存在细微差异，需要不断微调。
- 出现各种莫名其妙的行高、段落间距，调整过程非常麻烦。
- 无默认模版，对于写作新人来说，所写文章排版风格差异大，不利于阅读。

由于以上种种问题，会让我很难专注书写过程，在调整样式上浪费不少时间。

### 解决之道

首先我想到的就是 Markdown, 最近几年特别流行，自己的博客也一直用它书写，应该大家都很熟悉它吧。

它作为一门轻量的标记语言，能让你聚焦到内容本身，排版问题就让模版去解决，所以在很长一段时间我想做一款 Chrome 插件，能够直接将你书写的内容转化为带有微信支持的 HTML 代码。

不过，在我还未开发之前，已经在市面上找到了一款类似的工具，那就是 WXMarkdown: http://md.barretlee.com。

它基本满足了我的需求：

- 友好的代码高亮。
- 支持多种模版。
- 使用 Markdown 写作，转化的内容不存在样式偏差。

但它还有一些不足的地方：

- 不能够一键导入微信编辑器。
- 没有暂存功能，不敢在线编写，非常容易丢失内容。

### 总结

本篇文章使用了 WXMarkdown 转码，是不是样式看上去舒服很多，以后我会定期分享一些实用小工具，欢迎大家关注。