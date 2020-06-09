---
title: "对照 Ruby 学 Crystal - Part 0"
date: 2017-03-27 22:31:00 +0800
tags: [ruby]
---

Crystal 是一门神奇的语言，它号称 **Fast as C, slick as Ruby**，
至于速度方面有待考证，不过它的语法和 Ruby 非常相似，可见其胶水能力。

本篇文章是*对照 Ruby 学 Crystal* 系列基础篇，主要介绍 Crystal 社区中与 Ruby 对应的工具链。

### Version Manager

使用 [crenv](https://github.com/pine/crenv) 来管理多个不同版本的 Crystal， Ruby 中与之对应的是 rbenv 或者 rvm 。

### Dependency Manager

使用 [shards](https://github.com/crystal-lang/shards) 来管理项目依赖，Ruby 中对应的是 bundler 。


### Testing

测试框比较多，这里选取较常用的来做个比较，

[minitest.cr](https://github.com/ysbaddaden/minitest.cr) 对应 Ruby 中的 minitest 。
[spec2.cr](https://github.com/waterlink/spec2.cr) 对应 Ruby 中的 rspec 。

### Database Drivers

这个话题对应的就比较多了，应该主流的数据库都有对应的 driver 吧，例如，
[crystal-mysql](https://github.com/waterlink/crystal-mysql) 对应 Ruby 的 mysql2 。
[mongo.cr](https://github.com/datanoise/mongo.cr) 对应 Ruby 的 mongoid 。

### Web Framework

目前我的工作以 API 居多，所以这里选取了 [kemal](https://github.com/kemalcr/kemal), 它类似 Ruby 中的 sinatra 。

关于 Crystal 更多工具，可以参考 [awesome-crystal](https://github.com/veelenga/awesome-crystal), 这一个系列我会尝试一一展开介绍 Crystal 的具体使用，敬请期待。
