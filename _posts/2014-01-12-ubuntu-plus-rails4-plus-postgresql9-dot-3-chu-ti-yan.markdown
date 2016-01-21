---
layout: post
title: "Ubuntu + Rails4 + Postgresql9.3 初体验"
date: 2014-01-12 15:40:03 +0800
comments: true
share: true
category: technical
tags: [ror, postgresql]
---

> Postgresql9安装方法和步骤：

####1. 在  /etc/apt/sources.list.d/pgdg.list 文件中添加源

{% highlight bash%}
deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main
{% endhighlight%}

####2. 跟新软件源

{% highlight bash%}
sudo apt-get update
{% endhighlight%}
####3. 安装postgresql 9.3

{% highlight bash%}
sudo apt-get install postgresql-9.3
{% endhighlight %}
####4. 添加用户

{% highlight bash%}
sudo -u postgres psql postgres
{% endhighlight %}

####5. 设置密码

{% highlight bash%}
\password postgres
{% endhighlight %}

> Rails 相关配置


{% highlight ruby %}
gem 'pg'  #在gemfile中添加
{% endhighlight %}

修改database.yml

{% highlight ruby%}
# common config
 common: &common
   adapter: postgresql
   encoding: utf8
   pool: 10
   username: postgres
   password: password
   host: localhost

# development config
 development:
   database: kindergarten_development
   <<: *common

# test config
 test:
   database: kindergarten_test
   <<: *common

#production config
 production:
   <<: *common
   database: kindergarten

{% endhighlight %}
