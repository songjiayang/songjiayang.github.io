---
layout: post
title: "Ubuntu+Passager+Nginx  Rails部署搭建"
date: 2014-01-12 15:18:38 +0800
comments: true
share:  true
category: technical
tags: [passager, nginx]
---


跟新软件源

{% highlight bash%}
sudo apt-get update  .
{% endhighlight%}

如果升级报错，那么请执行sudo rm -rf /var/lib/apt/lists/* -vf 试试
安装git

{% highlight bash%}
sudo apt-get install git-core
{% endhighlight%}

安装curl

{% highlight bash%}
sudo apt-get install curl
{% endhighlight %}


安装rvm

{% highlight bash%}
curl -L https://get.rvm.io | bash -s
{% endhighlight %}
获得最新稳定版rvm和安装ruby的依赖

{% highlight bash%}
rvm get stable
rvm requirements
{% endhighlight %}
安装所有rvm requirements依赖

{% highlight bash%}
sudo apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion pkg-config
{% endhighlight %}
安装ruby

{% highlight bash%}
rvm install 2.0.0
rvm use 2.0.0 default
{% endhighlight %}
升级gem并设置gem的配置

{% highlight bash%}
gem update
echo -e "install: --no-rdoc --no-ri \ninstall: --no-rdoc --no-ri">> ~/.gemrc
{% endhighlight %}
安装Rials

{% highlight bash%}
gem install rails --version 3.2.13
{% endhighlight %}
安装Rails一些常见依赖

{% highlight bash%}
sudo apt-get install mysql-server mysql-client nodejs
{% endhighlight %}
安装passager/nginx

{% highlight bash%}
gem install passenger
rvmsudo passenger-install-nginx-module
{% endhighlight %}
所有选项选择默认即可.
nginx安装目录也选择默认 /opt/nginx/
passenger 会自动修改config, 添加passenger所在的rvm的ruby和gem环境地址

安装nginx脚本

{% highlight bash%}
wget https://raw.github.com/gist/1548664/53f6d7ccb9dfc82a50c95e9f6e2e60dc59e4c2fb/nginx
sudo cp nginx /etc/init.d/
sudo chmod +x /etc/init.d/nginx
sudo update-rc.d nginx defaults
{% endhighlight %}

配置nginx,文件在 /opt/nginx/config/nginx.conf

{% highlight bash%}
server {
   listen 80;
   server_name www.yourhost.com;
   root /home/railsu/project/public;   # <--- 这里是你项目的public目录
   passenger_enabled on;
}
{% endhighlight %}
启动nginx

{% highlight bash%}
sudo ./opt/nginx/sbin/nginx
{% endhighlight %}
重启项目

{% highlight bash%}
cd yourapp
touch tmp/restart.txt
{% endhighlight %}
