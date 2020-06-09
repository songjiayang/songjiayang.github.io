---
title: "如何将同一条告警信息发送给多个渠道"
date: 2018-01-19 23:55:26
tags: [prometheus]
---

> 背景： 有时我们需要将告警信息同时发送给多个渠道（例如短信或邮件），那么我们的 Alertmanager 该如何配置呢？

使用版本：Alertmanager 版本： 0.13.0

### 方法一

在同一个 recevier 定义多个接收渠道，例如：

```
route:
  receiver: my-receiver

receivers:
  - name: my-receiver
    webhook_configs:
    - url: 'https://hooks.xxx.com/xxxx'
    email_configs:
    - to: 'xx@xxxx'
      auth_username: 'xxx'
      auth_password: 'xxx'
```

说明: 可以看到同一条消息既使用 `webhook` 又使用 email 配置，所有在这两个渠道我们都收到消息。

### 方法二

```
route:
  receiver: email # 默认配置一个
  
routes:
  - match:
      severity: Critical
    continue: true
    receiver: webhook
    
 - match:
     severity: Critical
   receiver: email
   
receivers:
  - name: webhook
    webhook_configs:
    - url: 'https://hooks.xxx.com/xxxx'
  - name: email
    email_configs:
    - to: 'xx@xxxx'
      auth_username: 'xxx'
      auth_password: 'xxx'
```

定义多个独立的 receiver, 然后使用 routes 中的 `continue` 选项进行配置：

说明： 我们采用独立的两个 receiver 来接收消息，通过配置多个 routes 进行分发控制。

### 总结

我们可以使用以上两种方式实现同一条消息发送给不同渠道的效果，但是如果你的告警消息具有多类责任人（组），那么应该采用多个 routes 来分发消息，因为一个 receiver 代表了同一类接收者，这样配置也更灵活。