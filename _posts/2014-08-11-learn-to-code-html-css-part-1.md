---
title: Building Your First Web Page
date: 2014-08-11 22:31:00 +0800
tags: [html]
---

从今天起，我将根据逐步完成[Learn to Code HTML & CSS](http://learn.shayhowe.com/html-css/)系列课程的回顾学习，会将一些值得关注的知识点记录于此，留作笔记，以便日后查询。

### HTML部分

#### 1.1、基本概念

* Elements
* Tags
* Attributes

下图清晰的表明了它们具体指的是哪个部分：

![Html Terms](http://learn.shayhowe.com/assets/images/courses/html-css/building-your-first-web-page/html-syntax-outline.png)

#### 1.2、自关闭元素
```
1. <br>
2. <embed>
3. <hr>
4. <img>
5. <input>
6. <link>
7. <meta>
8. <param>
9. <source>
10. <wbr>

```

> 以上自关闭元素不需要使用元素结束符号 </ >

#### 1.3、引入样式表

一般项目中采用外部文件链接引入的方式：

```
<head>
  <link rel="stylesheet" href="main.css">
</head>
```

#### 1.4、HTML5简易模版

```
<!DOCTYPE html>
<html lang='zh-CN'>
  <head>
    <meta charset='utf-8'>
    <title>Title is here!</title>
    <link rel="stylesheet" href="main.css">
  </head>
  <body>
  </body>
</html>
```


### CSS部分

#### 2.1、基本概念

1. Selectors
2. Properties
3. Values

下图清晰的表明了三者之间的关联：

![](http://learn.shayhowe.com/assets/images/courses/html-css/building-your-first-web-page/css-syntax-outline.png)


#### 2.2、Selectors 不同类型

1. 元素选取
2. Class选取
3. Id选取
4. [复杂选取](http://learn.shayhowe.com/advanced-html-css/complex-selectors/)

#### 2.3、CSS Resets 来解决浏览器兼容性问题

```
/* http://meyerweb.com/eric/tools/css/reset/
   v2.0 | 20110126
   License: none (public domain)
*/

html, body, div, span, applet, object, iframe,
h1, h2, h3, h4, h5, h6, p, blockquote, pre,
a, abbr, acronym, address, big, cite, code,
del, dfn, em, img, ins, kbd, q, s, samp,
small, strike, strong, sub, sup, tt, var,
b, u, i, center,
dl, dt, dd, ol, ul, li,
fieldset, form, label, legend,
table, caption, tbody, tfoot, thead, tr, th, td,
article, aside, canvas, details, embed,
figure, figcaption, footer, header, hgroup,
menu, nav, output, ruby, section, summary,
time, mark, audio, video {
  margin: 0;
  padding: 0;
  border: 0;
  font-size: 100%;
  font: inherit;
  vertical-align: baseline;
}
/* HTML5 display-role reset for older browsers */
article, aside, details, figcaption, figure,
footer, header, hgroup, menu, nav, section {
  display: block;
}
body {
  line-height: 1;
}
ol, ul {
  list-style: none;
}
blockquote, q {
  quotes: none;
}
blockquote:before, blockquote:after,
q:before, q:after {
  content: '';
  content: none;
}
table {
  border-collapse: collapse;
  border-spacing: 0;
}
```

除了reset css ，你还可以选择[normalize.css](http://necolas.github.io/normalize.css/).

<hr>

课程链接：[http://learn.shayhowe.com/html-css/building-your-first-web-page/](http://learn.shayhowe.com/html-css/building-your-first-web-page/)
