---
title: "rails generate 命令的 orm 参数使用"
date: 2015-05-18 22:31:00 +0800
tags: [ruby]
---

当一个项目中出现多种数据库，例如 Postgresql 和 MongoDB ，对应的 gem 分别是 'pg' 和 'mongoid'.

此时使用 rails generate 命令默认生成的 model 将使用 `mongoid`，显然我们不想这么死板，最好有办法根据参数指定 model 所需类型。

这里我们可以使用 `--orm`  参数:

```

rails g modle post --orm=active_record
rails g modle post --orm=mongo_mapper

```

若不指定 orm 参数， 将优先使用 mongo_mapper 。

测试代码 [rails-orm](https://github.com/songjiayang/rails-example/tree/rails-orm) .
