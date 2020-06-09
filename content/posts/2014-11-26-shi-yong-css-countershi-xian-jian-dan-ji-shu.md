---
title: 使用 css counter 实现简单计数
date: 2014-11-26 22:31:00 +0800
tags: [css]
---

今天在 Matter 中添加了一个新功能，为 table 的每个 tr 添加一个序号， 大概效果如图：

![](http://songjiayang.qiniudn.com/demo.png)

由于需要在 table 中进行一些比较复杂的 dom 操作,比如 tr 的移动，删除，直觉告诉我，
使用js的方式会比较麻烦，所以就打算使用纯 css 方案。

#### 使用 css 的 [计数器](!https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Counters)

<p data-height="200" data-theme-id="10274" data-slug-hash="wBapNw" data-default-tab="result" data-user="songjiayang" class='codepen'>See the Pen <a href='http://codepen.io/songjiayang/pen/wBapNw/'>wBapNw</a> by 宋佳洋 (<a href='http://codepen.io/songjiayang'>@songjiayang</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

参考 https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Counters
