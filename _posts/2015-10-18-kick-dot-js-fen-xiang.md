---
layout: post
title: "Kick.js 分享"
category: technical
comments: true
share: true
description: ""
tags:
 - 开源
---

![kick.png](/images/kick.png)

[Kick.js](https://github.com/giga-opensource/kick.js) 是我们公司自己编写,并在项目中使用的一个 js 库。它其实就是在 Jquery 的 ajax 基础上
包装了一下，使其 json api 请求的写法更简单。

有了 kick.js, 你的请求可以写成这样：

#### 1. GET request

{% highlight javascript %}
Kick('materials').get()
# GET http://example.com/materials

Kick('materials').get({q:'panda'})
# GET http://example.com/materials?q=panda
{%endhighlight%}

#### 2. POST request
{% highlight javascript %}
Kick('materials').post({name: 'foo'})
# POST http://example.com/materials
# Request Payload
# {name: "foo"}
{%endhighlight%}

#### 3. PUT request
{% highlight javascript %}
Kick('materials', '1').put({name: 'foo'})
# PUT http://example.com/materials/1
# Request Payload
# {name: "foo"}
{%endhighlight%}

#### 4. PATCH request
{% highlight javascript %}
Kick('materials', '1').patch({name: 'foo'})
# PATCH http://example.com/materials/1
# Request Payload
# {name: "foo"}
{%endhighlight%}

#### 5. DELETE request
{% highlight javascript %}
Kick('materials', '1').delete()
# DELETE http://example.com/materials/1
{%endhighlight%}

源码中最值得分享的部分：

{% highlight javascript %}

var Kick = function() {
   if (!(this instanceof Kick)) {
     var instance = Object.create(Kick.prototype);
     return Kick.apply(instance, arguments);
   }

   # 此处省略部分代码

   return this;
 };
 {% endhighlight %}

点评：代码中， Kick 类作为方法调用，自动创建一个实例，避免了显示实例化的过程， 不然代码可能会写成 `(new Kick('materials')).get()` 。

目前，Kick.js 已提交到 github，欢迎大家使用。
