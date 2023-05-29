---
title: Rails 4.2 中很少有人知道的特性
date: 2014-08-31 22:31:00 +0800
tags: [ruby]
---

原文链接 [The Lesser-known Features in Rails 4.2](http://www.justinweiss.com/blog/2014/08/25/the-lesser-known-features-in-rails-4-dot-2/)  

[Rails 4.2 的第一个beta版本](http://weblog.rubyonrails.org/2014/8/20/Rails-4-2-beta1/)于上周发布了，这次引入的新的特性真的太酷了。想到马上就要在我的项目中使用 ActiveJob, Web Console, Adequate Record, 以及 Foreign Key support, 不免兴奋起来。

**但是Rails最美丽的地方是它的细节。** 如果多一点的挖掘，你可能会发现一些很少有人提到但却有可能极大提升你开发效率的新特性。

####更简单的配置文件加载

好了，可能我[有些偏见](http://www.justinweiss.com/blog/2014/05/27/using-a-little-bit-of-convenience-to-save-your-programming-flow/)。 但是拥有一个内置的配置文件加载将会变得更好！

`config_for`, 一个 Rails 4.2 的新特性，和你想的它的作用一样：


```
#config/redis.yml
development:
  host: localhost
  port: 6379
test:
  host: localhost
  port: 6379
production:
  host: redis-production
  port: 6379
```

```
irb(main):001:0> Rails.application.config_for(:redis)
=> {"host"=>"localhost", "port"=>6379}
```

也就是说， 如果你调用`config_for(:redis)`, 它将自动查找Rails项目中的`config/redis.yml` 文件，解析，并正确返回当前`RAILS_ENV`下的配置信息。

**它甚至可以让你在ymal中直接写ERB**

```
#config/redis.yml
development:
  host: localhost
  port: <%= ENV['REDIS_PORT'] %>
test:
  host: localhost
  port: <%= ENV['REDIS_PORT'] %>
production:
  host: redis-production
  port: <%= ENV['REDIS_PORT'] %>
```

```
$ REDIS_PORT=6380 bin/rails c
Loading development environment (Rails 4.2.0.beta1)
irb(main):001:0>  Rails.application.config_for(:redis)
=> {"host"=>"localhost", "port"=>6380}
```

如果你的项目有很多服务的配置信息，这有助于提高你的初始化配置信息的可读性。

#### 项目启动

大多数 Rails 项目在你启动他们之前，都需要你[执行一些命令](https://github.com/justinweiss/dgs_push_server/blob/master/setup.sh)， 即使那些空的Rails项目至少也会让你设置数据库。

所以，Rails， 再次遵从了*约定大于配置*的理念，创建了一个专门用于放置启动项目脚本的地方： `bin/setup`.

[默认的启动脚本已经不错了](https://github.com/rails/rails/blob/master/railties/lib/rails/generators/rails/app/templates/bin/setup)。但是 `bin/setup` 也是一个放置启动的代码的一个好地方。

所以，现在这已然成为了约定，**如果你的项目已经有一个启动脚本，那么请将它改名为 `bin/setup`** ； 如此，人们在使用你的项目的时候将变得更加简单。

`bin/setup` 能够使我们达成共识，少一些无聊的猜测。当你养成了 `git pull` 一个新项目后立马执行 `bin/setup`的习惯, 你就可以直接抛弃 `rake db:setup`
等设置命令了， :smile: 。

#### 变换 Hash 的值

这是另外一个我非常希望Ruby能够自带的功能。**当你在 Hash上调用 `transform_values` , 它就像在 Hash 的 values 上调用了 `map` 方法一样,返回一个新的 Hash ,这个 Hash 拥有和原来一样的 keys ,但是 values 却已经被替换了。**

```
h = {a: 1, b: 2, c: 3}

h.transform_values { |v| v * 2 } # => {a: 2, b: 4, c: 6}

```

够简单了吧, 但是你会惊讶为什么Rails会有这么多小的方便。


#### 额外：更多配置

Rails 4.2 提供了一种简单的方式来设置全局配置。

```
Rails.application.config.x.some_configuration_key = "Some Value"

Rails.application.config.x.some_configuration_key # => "Some Value"
Rails.configuration.x.some_configuration_key # => "Some Value"
```
如果你结合`config_for`， 它将工作的更好：

```
app_config = Rails.application.config_for(:app)
Rails.application.config.x.block_phone_calls = app_config["block_phone_calls"]
```

#### 更多

Rails 4.2 将是一个令人兴奋的发布版本。但是那些小的改进确保了 Rails 每天都变得更好。所以如果你还不太了解，那请快速浏览一下 [4.2 发布注意](http://edgeguides. rubyonrails.org/4_2_release_notes.html), 你将发现更多非常有用的改变。

(如果你发现其他很酷的功能，请不要保密，在此分享，如此，我们将会学习到更多新的东西)

**P.S. 我正在写一本关于毫无压力学习 Rails的书籍，更多详情，请点击 [这里](http://www.justinweiss.com/book/) 。**
