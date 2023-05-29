---
title: "Hello Rack"
date: 2014-11-25 22:31:00 +0800
tags: [ruby]
---

![http://rack.github.io/rack-logo.png](http://rack.github.io/rack-logo.png)

##### Rack 是什么？  
Rack 是一个模块化的 ruby webserver 接口；它是 web server 到 web framework的中间件。
##### 为什么需要 Rack?  
因为 Rack，项目可以从 web server 到 web framework 任意切换组合。例如同一个rails应用可以使用不同的
web server（unicorn, thin, puma ..）
##### Rack定义?

提供一个叫做 `app` 的 object， 这个 object 具有 `call` 方法；call 方法可以传递 ENV
作为参数; 返回是一个数组，数组包含了3个东西：

* http 返回状态码
* Hash 形式的 header信息
* 返回的body信息，必须具有 `each` 方法。

##### Rack 的基本使用

ruby 代码中直接运行

```
# my_rack_app.rb

require 'rack'

app = Proc.new do |env|
    ['200', {'Content-Type' => 'text/html'}, ['A barebones rack app.']]
end

Rack::Handler::WEBrick.run app
```  

使用rackup 命令行

```
# my.ru

run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['A barebones rack app.']] }
```


参考资料： http://rack.github.io/ ， https://github.com/rack/rack
