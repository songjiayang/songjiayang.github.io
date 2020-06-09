---
title: "Prometheus 数据安全之 Basic 认证"
date: 2018-01-30 22:56:26
tags: [prometheus]
---

我们都知道 Promethetheus 是不带认证和数据加密的，但很多时候我们又想在公网中直接使用它的管理界面，如果不做任何处理，它将存在严重的数据泄漏安全问题。

下面我将介绍如何通过 nginx 的 basic 认证来解决这一问题。

### 原始 Prometheus 代理配置

```
server {
    listen 80;
    server_name example.coom;

    location / {
        proxy_pass http://localhost:9090/;
    }
}
```

说明：使用 nignx 的 `proxy_pass` 代理我们 Prometheus 服务。

### 使用 htpasswd 生成认证密钥对

htpasswd 属于 `apache2-utils` 包，下面我将演示在 Ubuntu 上的安装和配置过程。

Step1: 安装 apache2-utils

```
$ sudo apt-get install apache2-utils
```

Step2: 生成认证密钥

```
$ htpasswd -c /etc/nginx/.htpasswd monitor


New password:
Re-type new password:
Adding password for user monitor
```

说明：`monitor` 可以修改为任意用户名。

Step3: 使用 cat 查看生成内容:

```
$ cat /etc/nginx/.htpasswd

monitor:$apr1$EEV......
```

### 修改 nginx 配置

```
server {
    listen 80;
    server_name example.coom;

    location / {
        auth_basic "Prometheus";
        auth_basic_user_file "/etc/nginx/.htpasswd";
        
        proxy_pass http://localhost:9090/;
    }
}
```

保存配置，重启 nignx 服务后，你访问域名 example.com 将出现 basic 认证的界面:

![nginx-basic.png](/images/prometheus/nginx-basic.png)

输入正确的账号密码，你将进入 Prometheus 管理界面。

### 写在最后

我们可以使用 nginx 的 basic auth 的方式来解决 Prometheus 认证的问题，此方法同样适用 Alertmanger 或者 NodeExporter。

从现在起，不要再让你的监控数据在公网中裸奔，赶快去修改配置吧。
