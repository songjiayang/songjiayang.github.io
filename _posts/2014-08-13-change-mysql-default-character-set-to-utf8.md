---
title: "How to change Mysql global character to utf8"
date: 2014-08-13 22:31:00 +0800
tags: [mysql]
---

最近我在做一个**讨论组**的应用，数据库是阿里云上自己安装的Mysql。今天突然发现有中文插入报错的情况，难道是数据库编码问题？

带着疑惑，登上服务器，进入mysql终端检查当前的编码,果不其然：

![](http://zuoyouba.qiniudn.com/blog_image_1.jpg)  
OK， 问题已经找到，那么开始解决。

#### 1、 查找本机mysql配置文件

由于服务器是linux系统，所以可以使用 `whereis` 命令来帮助寻找。

```
whereis mysql
mysql: /usr/bin/mysql /etc/mysql /usr/lib/mysql /usr/bin/X11/mysql /usr/include/mysql /usr/share/mysql /usr/share/man/man1/mysql.1.gz
```

最后发现我的mysql主配置文件是 `/etc/mysql/my.cnf` 。

#### 2、 修改配置文件，添加utf8相关设置

由于my.cnf文件中有这样一行代码 `!includedir /etc/mysql/conf.d/`, 所以我们可以将自定义配置信息放在`conf.d/`目录里面。这样的好处是方便配置模块化管理 , 个人比较推荐这样做。

新建/etc/mysql/conf.d/utf8.cnf 文件，并添加如下代码：

```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```

#### 3、 重启Mysql服务

```
sudo service mysql restart
```

至此Mysql全局的编码已经设置为了utf8，中文报错的问题已不复存在。


参考资料: [http://stackoverflow.com/questions/3513773/change-mysql-default-character-set-to-utf-8-in-my-cnf](http://stackoverflow.com/questions/3513773/change-mysql-default-character-set-to-utf-8-in-my-cnf)
