---
title: "RubyMotion 的 root path 另类使用姿势"
date: 2015-05-31 22:31:00 +0800
tags: [ruby]
---

在写 RubyMotion 代码过程中，不止一次获取项目的绝对路径的需求，解决办法都是根据当前文件，向上递推，再根据相对路径获取。

这样做费时又费力，代码还略显难看， 比如这样：

```
# app/lib/xx.rb
File.expand_path("../../models", __FILE__)
```

出现这个问题原因就是 RubyMotion 没有存储项目的根路径。 写 Rails 项目的人都知道，要想拿到 Rails 项目的根路径是非常容易的，只需调用 APP_NAME.root 即可。

那是否 RubyMotion 也可以这样呢？

下面是我的一个 Hack 方式：

打开 Rakefile 文件, 添加如下代码 :

```
# Rakefile

# Hack way to store app root path
ENV['APP_ROOT'] = File.expand_path("../", __FILE__)
```

然后你在项目中就可以通过 `ENV['APP_ROOT']` 得到项目根路径了。

JUST A JOKE , HAVA A FUN.
