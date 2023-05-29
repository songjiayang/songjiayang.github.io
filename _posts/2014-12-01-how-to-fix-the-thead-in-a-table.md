---
title: "how to fix the thead in a table only css"
date: 2014-12-01 22:31:00 +0800
tags: [css]
---

今天研究实现了一个将 table head 固定的效果。

主要思路是在 table 外包裹一个用于滚动的 `container`,然后设置其滚动样式：

```
.container {
  xxx
  overflow-x: scroll;
  overflow-y: hidden;
  xxx
}
```

然后再修改 tbody的滚动样式

```
.container {
  overflow-y: scroll;
}
```

最后 td, th 设置最小宽度即可。

完整代码请参考：

<p data-height="266" data-theme-id="10274" data-slug-hash="azOMGz" data-default-tab="result" data-user="songjiayang" class='codepen'>See the Pen <a href='http://codepen.io/songjiayang/pen/azOMGz/'>azOMGz</a> by 宋佳洋 (<a href='http://codepen.io/songjiayang'>@songjiayang</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>
