---
title: "瑞士军刀之 MotionModel 初体验"
date: 2015-05-31 23:31:00 +0800
tags: [ruby]
---

## 写在前面

[MotionModel](https://github.com/sxross/MotionModel) 是一个 RubyMotion 平台的 ORM 类库，有点类似于 Rails 中的 ActiveRecord。
它将业务模型与数据存储高度抽象，采用统一的调用方式，目前支持内存和数据库两种主要使用方式。

严格意义上来讲它能支持多种数据库，因为只需实现对应适配器即可。当然它已将我们常用的 Sqlite3 适配器写好了，所以可以直接使用。

我们项目数据存储使用的是 Sqlite3 + FMDB 的方案，所以将以此作为例子。


## 安装步骤

### 1. 修改 Gemfile 文件，添加 以下代码：

```
#Gemfile
gem 'motion_model', git:'https://github.com/sxross/MotionModel.git', branch:'sql'
```

说明： 之所以使用 `sql` 分支是因为目前项目已将数据库使用方式的代码剥离出来，还没打包合并到 master 分支，
所以如果你项目中使用了数据库，那就不能使用 master 分支， 详情请点击 [wiki](https://github.com/sxross/MotionModel/wiki/RoadMap#how-to-jump-on-board) 。

### 2. 执行 `bundle install`

### 3. 使用 cocoapods 引入 FMDB
修改 Rakefile 文件，添加 FMDB
```
#Rakefile
.....
app.pods do
  pod 'FMDB'
end
.....

```

修改文件后，执行 `rake pod:install`

### 4. 修改 Rakefile 文件， require motion_model 相关代码。

```
#Rakefile

require 'motion_model'
require 'motion_model/array' ＃ 如果不需内存存储方式，那么此句代码可以删除
require 'motion_model/sql'

```


## 具体使用

前提假设： 1. 所有 model 都使用 FMDB。 2. 目录结构是这样：

![motionmodel](/images/motionmodel.png)

### 1. 在 lib 文件下新建 一个 database.rb 文件用于初始化数据库链接并根据 model 自动创建表。代码如下：
```

# app/lib/database.rb

class Database
  def self.init
    MotionModel::Store.config(MotionModel::FMDBAdapter.new('app.db'))
    models = Dir.entries(File.expand_path("../../models", __FILE__)).select{ |file| file =~ /\.rb/ }.map{ |file| file.sub('.rb', '').capitalize.constantize }
    models.each { |clzss| clzss.create_table unless clzss.table_exists? }
  end
end
```

看到这段代码，有没有觉得 Ruby 文件操作和对象反射吊炸，处理初始化数据库真是再合适不过了， 好奇，不知 OC 方案是怎样。

### 2. 修改 app_delegate.rb， 在项目启动的时候附带启动数据库：
```
# app_delegate.rb
class AppDelegate
  def application(application, didFinishLaunchingWithOptions:launchOptions)
    Database.init
    true
  end
end

```


### 3. 在 app/models 目录下创建任意 model, 例如 user.rb
```
#user.rb

class User
  include MotionModel::Model
  include MotionModel::FMDBModelAdapter

  columns :name => :string,
          :email => :string,
          :created_at => :date,
          :updated_at => :date
end

```

当启动项目的时候，会先执行 Database.init, 此时会自动检查所有 model, 如果对应表没有创建，将自动创建。当然表的字段和配置的 columns 一致。

## MotionModel 的好处

1. 避免直接操作 sql， 例如我要查询所有学生，我可以写成 `User.all`;  查询名字是小明的学生 `User.where(:name => '小明')` 。
2. 更面向对象，支持模型之间的 RelationShip 。
3. 提供一些 validates, 例如 要求用户名不能为空,那么我们可以这样 `validates :name, :presence => true` 。

更多使用请参见官网。


## 写在最后：

其实我也是今天才开始使用 MotionModel 的，它给我的第一感觉就是太像 ActiveRecord , 在大概看了源码的基础上，决定先试试。
虽然不知道其 owner 会不会哪天弃坑，管它，到目前为止用的还顺心。欢迎有使用经验者以及有其他方案者多多交流经验。

<figcaption>写于: 2015/05/31 23:10 </figcaption>
