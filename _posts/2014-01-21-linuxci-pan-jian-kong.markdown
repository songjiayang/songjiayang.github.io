---
layout: post
title: "linux磁盘监控"
date: 2014-01-21 21:58:41 +0800
comments: true
share: true
category: technical
tags: [linux]
---

大家可能都知道Linux上的`top`命令，使用它可以实时显示系统的运行状况，如图:

![top](http://book-share.qiniudn.com/Screenshot%20from%202014-01-21%2022:01:34.png)

top命令很好，唯独无法清楚的监控系统磁盘I/O的读写情况，iotop这个软件就可以很好的弥补top不足，能对服务器的磁盘进行实时监控。

#### iotop安装
{% highlight bash %}
 sudo apt-get install iotop
{% endhighlight %}

####  运行iotop
{% highlight mysql %}
sudo iotop  
{% endhighlight %}

#### 运行效果

![](http://book-share.qiniudn.com/Screenshot%20from%202014-01-21%2022:01:56.png)
