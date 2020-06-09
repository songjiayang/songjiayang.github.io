---
title: "Bundle 的时候出现 'unable to connect to github.com' 问题"
date: 2014-11-07 22:31:00 +0800
tags: [ruby]
---

最近新开了一台日本的亚马逊机器，在使用 cap2 做发布的时候， 在 bundle 那步出现报错，提示信息类似：

```
Fetching git://github.com/activeadmin/activeadmin.git
fatal: unable to connect to github.com:
github.com[0: 192.30.252.130]: errno=Connection refused
```

这是因为我部分 gem 是使用 github 仓库安装，而服务器上无法通过 git 协议连接到 github 获取代码, 所有报错。出现这个原因可能是因为我亚马逊机器设置的问题，（比如某些协议不支持或者端口没打开）。

Anyway, 既然 git 协议通不过，那么我们可以尝试换成 https 协议。遗憾的是，Gemfile里面，不能直接指定请求发送的协议， 但是我们可以尝试修改 git global config ，针对 github 将 git 协议重定向到 https上，这个有点像 dns 设置。

具体设置:

```
git config --global url."https://github.com".insteadOf git://github.com
```

在服务器上执行了以上代码后,重新执行 `cap production deploy` 就可以了。

参考链接：
http://stackoverflow.com/questions/21260689/force-bundle-install-to-use-https-instead-of-git-for-github-based-gems
