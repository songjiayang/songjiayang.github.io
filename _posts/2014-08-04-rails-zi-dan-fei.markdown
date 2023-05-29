---
title: Newrelic 让你的Rails子弹飞
date: 2014-08-04 22:31:00 +0800
tags: [ruby]
---

![子弹飞](/images/zidanfei.png)

[Newrelic](http://newrelic.com)是一个web应用程序性能监控工具；它功能强大，界面友好，完全免费，好处多多，谁用谁知道（无耻的打了下广告）。

### Rails中安装Newrelic

#### 1. 注册Newrelic账户

Newrelic账户完全免费，请前往http://newrelic.com/ 注册吧。

#### 2. 新建监控应用，并生成license key，并下载自动生成的newrelic.yml

![生成监控应用](/images/newrelic-setup-app.png)

####3. 使用bundler安装New Relic代理 ####

在gemfile文件添加：

```
gem 'newrelic_rpm'
```

执行bundle:

```
bundle install
```

#### 4. copy下载的newrelic.yml到config/newrelic.yml

修改newrelic.yml中`app_name:xxx`这一样，将其修改为自己项目名称，这个名称会用于newrelic后台监控项目查询。

#### 5. 重新发布应用

当网站重新发布后，进入newrelic的web后台，选择刚发布的应用，将看到类似统计图表：
![生成监控应用](/images/newrelic-chat.png)

Newrelic的监控后台是实时的， 包含了Middleware和Database的监控，其图表显示的访问速度，对于网站优化很有意义，值得一试。
