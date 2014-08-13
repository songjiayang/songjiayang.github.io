---
layout: post
title: "Mysql Wiki"
date: 2014-01-21 15:07:16 +0800
comments: true
category: archive
tags: [mysql]
---


1.1 Drop a colomn form a table

``` mysql
 ALTER TABLE tb_name DROP COLUMN column_name ;
```

1.2 Give a user all permission to a table

``` mysql
GRANT ALL PRIVILEGES ON dbTest.* To 'user'@'hostname' IDENTIFIED BY 'password';
```
1.3 Change databse charset to utf-8：

``` mysql
ALTER DATABASE databasename CHARACTER SET utf8 COLLATE utf8_unicode_ci;
ALTER TABLE tablename CONVERT TO CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

1.4 Install mysql in rails

``` bash
sudo apt-get install mysql-server-xx
sudo apt-get install libmysqlclient-dev 
gem install mysql2
```
