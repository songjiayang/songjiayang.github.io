---
layout: post
title: "Mysql Wiki"
date: 2014-01-21 15:07:16 +0800
comments: true
share: true
category: technical
tags: [mysql]
---


#### Drop a colomn form a table

{% highlight mysql %}
ALTER TABLE tb_name DROP COLUMN column_name ;
{% endhighlight %}

#### Give a user all permission to a table

{% highlight mysql %}
GRANT ALL PRIVILEGES ON dbTest.* To 'user'@'hostname' IDENTIFIED BY 'password';
{% endhighlight %}
#### Change databse charset to utf-8：

{% highlight mysql %}
ALTER DATABASE databasename CHARACTER SET utf8 COLLATE utf8_unicode_ci;
ALTER TABLE tablename CONVERT TO CHARACTER SET utf8 COLLATE utf8_unicode_ci;
{% endhighlight %}

#### Install mysql in rails

{% highlight mysql %}
sudo apt-get install mysql-server-xx
sudo apt-get install libmysqlclient-dev
gem install mysql2
{% endhighlight %}
