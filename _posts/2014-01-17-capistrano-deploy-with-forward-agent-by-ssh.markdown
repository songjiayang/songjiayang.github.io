---
layout: post
title: "Capistrano Deploy With Forward Agent By SSH"
date: 2014-01-17 12:01:51 +0800
comments: true
category: archive
tags: [tool, deploy]
---

使用Capistrano部署应用的时候，服务器往往会到一个具有访问权限的git仓库拉取代码; 如果你不想加入server的.ssh/id_rsa.pub到仓库的拉取权限组，那么你可以尝试使用本地代理的方式。

具体操作如下：

#####1. 配置deploy文件,手动开启代理服务.

	{% highlight ruby %}
	set :ssh_options, {:forward_agent => true}
	{% endhighlight %}

#####2. 告诉SSH Agent你的key

	{% highlight ruby%}
	$ ssh-add ~/.ssh/id_rsa
	{% endhighlight %}


 参考资料

 * [http://opensoul.org/2009/06/24/capistrano-git-and-ssh-keys/](http://opensoul.org/2009/06/24/capistrano-git-and-ssh-keys/)
