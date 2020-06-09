---
title: "如何定制微信告警模版"
date: 2018-03-01 04:11:07
tags: [prometheus]
---

AlertManger 从 v0.12 已经默认支持企业微信发送告警信息了，具体步骤可以参见 [alertmanger-with-wechat](http://www.songjiayang.com/posts/alertmanger-with-wechat)。

最近不少朋友在群里问到：“如何定制微信告警消息？”，下面来讲解下具体步骤。

### 1. 添加自定义模版

在 alertmanager 目录添加 一个 `wechat.tmpl` 文件，根据需求添加告警信息。

a: 如果分组有多条告警信息，可以通过循环输出的方式：

<script src="https://gist.github.com/songjiayang/25e43aefabbfab204e54c382144eb3ba.js"></script>

效果如图：

![weixin02.png](/images/weixin02.png)

b: 如果分组只想显示一条信息，那么可以使用公用信息：
<script src="https://gist.github.com/songjiayang/4c5bd6f6908f96839fe4e7e88bd0c733.js"></script>

效果如图：

![weixin03.png](/images/weixin03.png)

### 2. 修改配置文件

打开 `alertmanger.yml` 文件，添加自定义模版配置：

```
templates:
  - 'wechat.tmpl'
```

保存重新加载配置即可。
