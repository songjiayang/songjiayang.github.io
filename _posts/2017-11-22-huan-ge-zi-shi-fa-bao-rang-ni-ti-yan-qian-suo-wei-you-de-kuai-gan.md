---
title: "换个姿势发包，让你体验前所未有的快感"
date: 2017-11-22 23:39:21
tags: [效率]
---

一种全新的方式进行 Go 语言项目打包并发布到 GitHub, 让你体会到前所未有的快感。

### 背景

我们都知道 Go 语言项目非常方便交叉编译，这意味着 GitHub 项目在发布新版本的时候，
需要尽可能编译成不同平台的二进制文件，并再将它们一一上传到此次 release 的下载列表中，
整个过程显得重复和耗时。

下面我将介绍使用 gco 和 ghr 将整个过程自动化起来，实现一键打包发布， 其中 gco 负责编译， ghr 负责上传发布。

### gco 使用

`gco` 是我写的一个针对 Go 项目的交叉编译脚本。

安装步骤

```
curl https://raw.githubusercontent.com/songjiayang/gco/master/gco.sh -o  /usr/local/bin/gco

chmod +x /usr/local/bin/gco
```

使用方法

```
gco example v0.1 main.go
```

参数说明：

- example 是你编译的项目名称
- v0.1 是编译的版本
- main.go 是项目的 main 包位置

所有编译的包默认放到项目的 ./pkg 目录 下。

### ghr 使用

ghr 是社区写的一个批量上传打包文件到 GitHub 的命令行工具，你可以到 https://github.com/tcnksm/ghr/releases 下载对应版本。

使用方法：

```
export GITHUB_TOKEN="....." // 添加 GITHUB_TOKEN 环境变量

ghr v0.1 pkg/
```

参数说明：

- v0.1 是发布的版本
- pkg/ 是为二进制包放的目录

### 实战

下面是我录制的一段小视频，根据一个实际例子演示整个发布流程，参考 [链接](https://mp.weixin.qq.com/s?__biz=MzA5ODg4ODY2OA==&mid=2648014953&idx=1&sn=e7ad5f952e9c4445921ee7defaf3c312&chksm=88ab9edabfdc17cc89da8e124582d831043e8cd7935221c690406daa36cf513630d2862f7938#rd) 。
