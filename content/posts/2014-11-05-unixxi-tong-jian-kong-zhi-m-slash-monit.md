---
title: "超好用的监控工具之 M/Monit"
date: 2014-11-05 22:31:00 +0800
tags: [linux]
---

#### 1. [M/Monit](http://mmonit.com/) 是什么？

简单而言，Monit 是一款轻量级，好用的监控工具，它能够监控unix系统，网络，云服务等任务，它还能够自动重启服务。<br/>
下面是一段官方的描述：
> Easy, proactive monitoring of Unix systems, network and cloud services. Conduct automatic maintenance and recovery and execute meaningful causal actions in error situations


#### 2. 如何安装 M/Monit ?

apt 类系统上安装：
```
sudo apt-get install monit
```

yum 类系统上安装:
```
sudo yum install monit
```

<!-- 源码安装（下载地址： http://mmonit.com/download/），以 linux-x64 为例：

```
wget http://mmonit.com/dist/mmonit-3.3-linux-x64.tar.gz
tar -xf mmonit-3.3-linux-x64.tar.gz & cd mmonit-3.3
sudo ./config & make & make install
``` -->

#### 3. M/Monit 主要目录

以ubuntu系统为例，monit的配置目录主要在  `/etc/monit`, 主要包含：

```
drwxr-xr-x  2 root root  4096 Nov  5 05:33 conf.d
-rw-------  1 root root 10962 Nov  5 04:30 monitrc
drwxr-xr-x  2 root root  4096 Nov  4 10:24 monitrc.d
drwxr-xr-x  2 root root  4096 Nov  4 10:24 templates
```

其中 monitrc 是主配置文件，config.d 是各种具体任务监控的配置目录，monitrc 会包含 conf.d 所有文件。 monitrc.d 里面有很多配置模版，你可以直接使用，只需要做个软链到 conf.d 即可。

#### 4. 一些常用任务监控脚本

nginx 监控

```
 check process nginx with pidfile /var/run/nginx.pid
   group www
   group nginx
   start program = "/etc/init.d/nginx start"
   stop program = "/etc/init.d/nginx stop"
#  if failed port 80 protocol http request "/" then restart
   if 5 restarts with 5 cycles then timeout
   depend nginx_bin
   depend nginx_rc

 check file nginx_bin with path /usr/sbin/nginx
   group nginx
   include /etc/monit/templates/rootbin

 check file nginx_rc with path /etc/init.d/nginx
   group nginx
   include /etc/monit/templates/rootbin
```

redis 监控

```
if failed port 6379 then alert
```

系统资源监控

```
check system matter.build
  if loadavg (1min) > 4 then alert
  if loadavg (5min) > 2 then alert
  if memory usage > 75% then alert
  if swap usage > 25% then alert
  if cpu usage (user) > 70% then alert
  if cpu usage (system) > 30% then alert
  if cpu usage (wait) > 30% then alert

check device var with path /
  if SPACE usage > 80% then alert
```

sidekiq 监控

```
check process rails_app-sidekiq
   with pidfile /var/www/rails_app/shared/pids/sidekiq.pid
   stop program = "/usr/bin/env HOME=/home/deploy /bin/bash -c '[ ! -f /var/www/rails_app/current/tmp/pids/sidekiq.pid ] || kill `cat /var/www/rails_app/current/tmp/pids/sidekiq.pid` || true'"
   if changed pid then alert
```

unicorn 监控

```
 if failed unixsocket /tmp/unicorn.rails_app_production.sock then alert
```
