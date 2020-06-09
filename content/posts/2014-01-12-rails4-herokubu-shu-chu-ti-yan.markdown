---
title: "Rails4 + Heroku 部署初体验 "
date: 2014-01-12 15:22:23 +0800
tags: [ruby] 
---

想必做Rails 开发的很少有人不知道[heroku](heroku.com)的吧,它其实是一个pass平台。操作简单，文档齐全，唯一不好的就是国内访问速度比较慢（不过，人家是免费的哦）。

**Q** ： 到底使用heroku部署一个Rails 项目有多容易？  
**A** ： 真的太容易了.

### 1. 注册heroku帐号

### 2. 安装heroku gem  

```
gem install heroku
```

### 3. 配置Gemfile
```
group :production do
	gem 'rails_12factor'
	gem 'pg'
end
```

### 4. 发布到heroku

```
heroku login
heroku create
git add .
git commit  -m  "for heroku"
git push heroku master
```  

####5. heroku 一些常用命令
```
heroku run rake db:migrate
heroku run rake db:seed
```

每个项目都会自动生成一个免费的亚马逊的postgresql数据库用户，可以到heroku 控制台查询。需要特别指出这里不需要执行db:create等命令，因为数据库已经存在，不然会报出权限相关的错误。
