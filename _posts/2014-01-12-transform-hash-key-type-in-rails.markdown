---
layout: post
title: "Transform Hash Key Type In Rails"
date: 2014-01-12 15:11:08 +0800
comments: true
category: archive
tags: [rails] 
---


Symbol To String  

{% highlight ruby %}
foo = {
    :a => 1,
    :b => 2
}

foo.with_indifferent_access
=>
{
    "a" => 1,
    "b" => 2
}

{% endhighlight %}


String To Symbol 

{% highlight ruby %}
foo = {
    "a" => 1,
    "b" => 2
}

foo.symbolize_keys
=>
{
    :a => 1,
    :b => 2
}

{% endhighlight %}


