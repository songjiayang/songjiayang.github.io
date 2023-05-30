---
title: "如何在本地查看 GitBook 电子书"
date: 2018-02-28 06:19:38
tags: [tools]
---

### 背景

至从在 gocn 发布 [《Go 零基础编程入门教程》](https://gocn.io/question/1615)的新闻后，越来越多的小伙伴加入到了我们的 go 语言学习群(QQ:694650181)。

在学习的过程中，有不少同学都在反馈配套电子书 [GitBook](https://songjiayang.gitbooks.io/go-basic-courses/content)在线无法打开的问题（因为 gitbooks.io 被墙）。

所以今天我要介绍的是如何使用 gitbook 命令行工具在本地查看 GitBook 电子书，下面我就以《Go 零基础编程入门教程》为例。

### 安装 gitbook

使用 npm 安装：

```
npm install gitbook-cli -g
```

参考 [本地安装 gitbook 命令行](https://git.io/vAXt7)

### 下载电子书源代码并在本地查看

a. 下载电子数

```
git clone git@github.com:songjiayang/go-basic-courses.git
```

b. 运行 gitbook 服务

```
cd go-basic-courses
gitbook serve
```

然后打开浏览器，访问 `http://localhost:4000` 即可。

![gitbook.png](/images/gitbook.png)
