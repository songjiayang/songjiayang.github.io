---
layout: post
title: "Linux下磁盘性能简单检测"
date: 2014-01-28 20:30:48 +0800
comments: true
share: true
category: technical
tags: [linux]
---

1.使用dd命令简单测试磁盘I/O

{% highlight bash %}
dd if=/dev/zero of=testfile bs=1M count=512 conv=fdatasync
{% endhighlight %}
2.使用hdparm命令

{% highlight bash %}
sudo hdparm -t /dev/sda
{% endhighlight %}
