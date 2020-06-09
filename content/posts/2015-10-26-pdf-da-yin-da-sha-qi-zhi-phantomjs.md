---
title: "PDF 打印大杀器之 PhantomJS"
date: 2015-10-26 23:31:00 +0800
tags: [ruby]
---
![phantomjs.png](/images/phantomjs.png)

PDF 打印是件麻烦事 ！！！

#### 开发调试问题：
1. 大量代码与样式，输出内容排版组织有关，单元测试难写 。
2. 没有简单的预览功能，无法逐步调整样式 。

#### 定制样式问题：
1. PDF 本身支持的内容样式十分多，所以通过程序 DSL 构件 PDF 的代码冗余高 。
2. 图片排版始终是个问题 。
3. 国际化的问题（有些库对 I18n支持不够友好）。

#### 为什么选择 PhantomJS ？
1. PhantomJS 的屏幕抓取功能十分好用，PDF 打印支持好，抓取打印代码简单 。
2. 内容模版使用 html+css，代码简单而且方便预览调试，开发更容易 。

#### PhantomJS 打印的例子

```
// Simple Javascript example

console.log('Loading a web page');
var page = require('webpage').create();
var url = 'http://phantomjs.org/';
page.open(url, function (status) {
  //Page is loaded!
  page.render('example.pdf')
  phantom.exit();
});
```

应该各语言都有它的 wrapper , Ruby 语言我推荐 [phantompdf](https://github.com/mcspring/phantompdf),
因为公司开源项目，而且我们项目中一直在用 :smile: 。

总结： 如果你还在为项目中 PDF 打印而烦恼，不妨尝试使用 PhantomJS 的方案，就个人经验而言，它是我见过 PDF 打印最简单的方式了。
