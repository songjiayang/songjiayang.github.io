---
layout: post
title: "shipped redmine on unicorn and nginx"
category: archive
description: "让你的 redmine 跑在 unicorn nginx 上 "
tags:
 - deploy
 - software
---

![tools , we need to choose](http://songjiayang.qiniudn.com/stock-photo-vintage-tools-workshop-91655433.jpg)
#### 背景

上午偶然看到 xdite 的一篇博客，[浅谈软体专案管理，如何挑选工具与方法](http://blog.xdite.net/posts/2015/02/18/introduction-project-management)
，才知道它们也在使用 redmine 进行项目管理，而且用的很有心得的样子。

下午 PM 找到我，和我一起讨论了新的一年在项目管理和工作效率方面可以改进的地方，主要包括了 workflow 和 tools 两方面。
我们一直是使用 Trello 作为看板管理项目的；但个人觉得，当项目的 workflow 变得复杂以后，Trello 使用起来就没有那么爽了。
不同角色的干扰信息太多，信息统计方面也不够好，不够直观。

刚好，加拿大团队使用的是 redmine 来管理另外一个的项目，所以我和PM商量，在国内的服务器上也安装个 redmine 。

#### 写在前面

由于官方的安装教程已经非常详细，具体安装过程我就不再一一赘述，一步一步跟下来即可。本文主要想讲的是如何通过简单配置，让你的 redmine 跑在 nginx 和 unicorn 这样的组合上。

#### 修改配置

##### 1. 在 `Gemfile.local` 中添加 `gem 'unicorn' `，执行 bundle 。
##### 2. 新建 config/unicorn.rb 文件，添加内容大致是：

```ruby
# working_directory "/path/to/your/app"
working_directory "/var/www/redmine"

# Unicorn PID file location
# pid "/path/to/pids/unicorn.pid"
pid "/var/www/redmine/tmp/pids/unicorn.pid"

# Path to logs
# stderr_path "/path/to/log/unicorn.log"
# stdout_path "/path/to/log/unicorn.log"
stderr_path "/var/www/redmine/log/unicorn.log"
stdout_path "/var/www/redmine/log/unicorn.log"

# Unicorn socket
listen "/tmp/unicorn.redmine.sock"

# Number of processes
# worker_processes 4
worker_processes 2

# Time-out
timeout 30
```
##### 3. 添加一个 nginx server:  

```nginx
upstream redmine {
  server unix:/tmp/unicorn.redmine.sock;
}

# HTTP
server {
  listen 80;
  server_name redmine.gigabase.org;

  root /var/www/redmin/public;

  gzip on;

  client_max_body_size 20M;
  error_page 404 /404.html;
  error_page 500 /500.html;

  location / {
    proxy_set_header Vary Accept;
    proxy_set_header X-UA-Compatible IE=edge,chrome=1;
    proxy_set_header X-Real-IP  $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X_FORWARDED_HOST $http_host;
    proxy_set_header X-Sendfile-Type X-Accel-Redirect;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    if (!-f $request_filename) {
      proxy_pass http://redmine;
      break;
    }
  }
}
```

##### 4. 重启服务:

```bash
sudo nginx -s reload
unicorn_rails -c /var/www/redmine/config/unicorn.rb -E production -D
```

#### 总结：

Redmine 的安装和适配是非常容易的，作为一款出道多年的开源项目管理软件，值得大家试试。
