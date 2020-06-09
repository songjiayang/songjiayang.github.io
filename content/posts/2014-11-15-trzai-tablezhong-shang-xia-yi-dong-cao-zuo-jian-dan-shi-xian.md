---
title: "Table 中任意上下移动 row 的简单实现"
date: 2014-11-15 22:31:00 +0800
tags: [css]
---

刚在 matter.build 里面实现了一个给项目材料任意排序的功能，主要技术包含：

* table中如何实现上下移动
* sectionRowIndex 属性使用

##### Row 上下移动主要使用使用jquery 的 prev,next 和 after,before 来实现,具体代码如下：

```
$.fn.moveUp = ->
  $.each this,  ->
    $(this).after $(this).prev()

$.fn.moveDown = ->
  $.each this,  ->
    $(this).before $(this).next()
```

有了上面代码，你就可以通过 `$('tr::eq(2)').moveUp()` 和 `$('tr::eq(0)').moveDown()`
来实现 tr上下移动了。

##### 想获取任意一个tr在table中位置，可以使用 tr 的 [sectionRowIndex](http://www.w3schools.com/jsref/prop_tablerow_sectionrowindex.asp) 属性 。
