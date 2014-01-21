---
layout: post
title: "Capistrano Deploy With Forward Agent By SSH"
date: 2014-01-17 12:01:51 +0800
comments: true
categories: [ssh, capistrano, agent]
---

使用Capistrano部署应用的时候，服务器往往会到一个具有访问权限的git仓库拉取代码，对于此，如果你不想加入server的.ssh/id_rsa.pub到仓库的拉取权限组，那么你可以尝试使用本地代理的方式。

具体操作如下：

1.配置deploy文件,手动开启代理服务. 

	{% codeblock lang:ruby deploy.rb %}
	set :ssh_options, {:forward_agent => true}
	{% endcodeblock %}

2.告诉SSH Agent你的key  

	{% codeblock lang:ruby console%}
	$ ssh-add -K
	{% endcodeblock %}


 参考资料

 * http://opensoul.org/2009/06/24/capistrano-git-and-ssh-keys/
