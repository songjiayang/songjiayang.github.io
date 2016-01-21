---
layout: post
title: "Javascript中如何忽略大小正则匹配动态字符串"
category: technical
tags:
 - javascript
---

大家应该都知道在 Javascript 中正则匹配可以使用 [match](http://www.w3schools.com/jsref/jsref_match.asp) 方法。

一个最简单的正则匹配例子 ：

{% highlight javascript%}
 "Foo".match(/foo/i);
{% endhighlight %}

输出结果：
{% highlight ruby%}
['Foo']
{% endhighlight %}

Ok, 要解决标题中所说问题，我有必要先说明下在这里动态字符串指的是个什么？

动态字符串是指： 正则表达式是个变量，具体内容无法提前知道，例如需要对用户输入的结果进行正则匹配。那该如何处理这种情况？

我们可以通过实例化一个带有字符串参数的 RegExp 对象来处理这个问题，例如：

{% highlight javascript%}
var input = "foo" ;
var re = new RegExp(input, "i") ;
"Foo".match(re) ;
{% endhighlight %}

输出结果：
{% highlight javascript%}
['Foo']
{% endhighlight %}

代码中 input 变量表示用户输入的值（可以根据具体情况获取）， re 就是根据输入内容生成的 RegExp 实例，其中 `i` 这个参数表示忽略大小写。

<hr>

参考资料：  

1. [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match)
2. [http://www.w3schools.com/jsref/jsref_obj_regexp.asp](http://www.w3schools.com/jsref/jsref_obj_regexp.asp)
