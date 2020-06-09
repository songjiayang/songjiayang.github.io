---
title: "AlertManager 邮件告警配置详解"
date: 2018-01-22 21:56:26
tags: [prometheus]
---

邮件作为最为通用的告警手段，我想大家在平时的工作中一定不会少用，下面我就以 gmail 为例来讲解整个配置过程。

版本：Alertmanager 版本: 0.13.0

### 配置内容

修改 `alertmanager.yml` 文件，添加信息如下：

```
route:
  group_by: [Alertname]
  receiver: email 
  
receivers:
  - name: 'email'
  email_configs:
  - smarthost: 'smtp.gmail.com:587'
    hello: 'gmail.com'
    from: 'xx@gmail.com'
    to: 'xx@gmail.com'
    auth_username: '' // email 账号
    auth_identity: '' // email 账号
    auth_password: '' // email 密码
```

如果你有多个 email recevier，而它们使用相同的 smtp sever，那么我们可以使用全局 smtp 来简化配置：

```
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'xx@gmail.com'
  smtp_hello: 'gmail.com'
  
receivers:
  - name: 'email'
  email_configs:
  - to: 'xx@gmail.com'
    auth_username: '' // email 账号
    auth_identity: '' // email 账号
    auth_password: '' // email 密码
```

重要提醒：

- auth_password 为 gmail 密码，通常你需要的是一个 App Password。
- 尽量避免使用非专业邮箱发送告警，例如 QQ 邮箱。

更新配置后，如果有新的告警信息，你将收到类似邮件：

![email-alert.png](/images/prometheus/alertemail.png)

### 自定义消息模版

告警通知使用的是默认模版，因为它已经编译到二进制包了，所以我们不需要额外配置。如果我们想自定义模版，这又该如何配置呢？

步骤一： 下载官方默认模版

```
$ wget https://raw.githubusercontent.com/prometheus/alertmanager/master/template/default.tmpl
```

步骤二： 根据自己的需求修改模版，主要是下面这一段

```
define "email.default.html" 
.... // 修改内容
end 
```

步骤三: 修改 alertmanger.yml，添加 templates 配置参数

```
templates:
- './template/*.tmpl' //  自定义模版路径
```

最后保存重新加载配置即可。