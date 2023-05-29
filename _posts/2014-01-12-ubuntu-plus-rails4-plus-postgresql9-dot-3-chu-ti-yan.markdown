---
title: Ubuntu + Rails4 + Postgresql9.3 初体验
date: 2014-01-12 15:40:03 +0800
tags: [ruby] 
---

> Postgresql9安装方法和步骤：

#### 1. 在  /etc/apt/sources.list.d/pgdg.list 文件中添加源

```
deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main
```

#### 2. 跟新软件源

```
sudo apt-get update
```
#### 3. 安装postgresql 9.3

```
sudo apt-get install postgresql-9.3
```
#### 4. 添加用户

```
sudo -u postgres psql postgres
```

#### 5. 设置密码

```
\password postgres
```

> Rails 相关配置


```
gem 'pg'  #在gemfile中添加
```

修改database.yml

```
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

```
