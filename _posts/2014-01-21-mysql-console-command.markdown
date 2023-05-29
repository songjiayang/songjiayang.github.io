---
title: Mysql Wiki
date: 2014-01-21 15:07:16 +0800
tags: [mysql] 
---


#### Drop a colomn form a table

```
ALTER TABLE tb_name DROP COLUMN column_name ;
```

#### Give a user all permission to a table

```
GRANT ALL PRIVILEGES ON dbTest.* To 'user'@'hostname' IDENTIFIED BY 'password';
```
#### Change databse charset to utf-8：

```
ALTER DATABASE databasename CHARACTER SET utf8 COLLATE utf8_unicode_ci;
ALTER TABLE tablename CONVERT TO CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

#### Install mysql in rails

```
sudo apt-get install mysql-server-xx
sudo apt-get install libmysqlclient-dev
gem install mysql2
```
