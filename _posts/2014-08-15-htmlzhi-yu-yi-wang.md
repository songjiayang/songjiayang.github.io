---
title: "HTML 之语义网"
date: 2014-08-15 22:31:00 +0800
tags: [html]
---

#### 什么是语义网

语义网的本质就是根据内容使用正确的元素， 什么样的元素就该对应什么样的页面内容，而不应该考虑样式的问题。

例如文章标题，不应该是这样：

```
<font size="6"><b>This is the page title</b></font>
```

而应该是这样：

```
<h1>This is a heading</h1>
```

其实几乎所有HTML元素都是具有语义的， 除`<div>`, `<span>`以外。它们没有具体的语义，仅仅是为了方便样式表使用而已。

#### 语义网的好处

1. 代码更简短，从而让用户下载速度更快。
2. 代码更易响应变化，方便样式/内容改版， 因为你的内容和样式是分离的，。
3. 代码更易人阅读，方便他人学习和接手。
4. 更方便爬虫程序理解，有助于搜索排名。

更多好处，请参考[http://boagworld.com/dev/semantic-code-what-why-how/](http://boagworld.com/dev/semantic-code-what-why-how/)

#### 如何保证你的网站是语义的

现在还没有工具能够自动检测网站是否满足语义。但是我们是能够随着经验和认识的提高不断优化我们的代码，实现最终的完全语义。

其实HTML5的出现，也大大促进了语义网的概念，它新增的元素都非常有用而富有语义。

一个好的实践：

![HTML5 语义网实践](http://learn.shayhowe.com/assets/images/courses/html-css/getting-to-know-html/building-structure.png)

> 既然语义网如此重要，我们没有理由不用，大家以后不妨试试。

参考资料: [http://learn.shayhowe.com/html-css/getting-to-know-html/](http://learn.shayhowe.com/html-css/getting-to-know-html/)
