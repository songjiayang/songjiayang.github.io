---
title: "使用 which-better 帮助你挑选更好的依赖库"
date: 2018-01-26 23:55:58
tags: [opensource]
---


作为一名职场新人对于工具或者开源库的选择往往会陷入长时间的纠结，因为你不知道选择哪个更好。如果没有老司机的指导，一步小心你就会 make a bad chose。

基于此，我开发了一个小工具来帮助大家，用它会大大缩短你选择时间，而且不会出现大问题。

### 前言

- 我们选择的是具有相似功能的库或者系统。
- 备选方案都放在 GitHub，因为 GitHub 是最火的开源软件平台。

### 工作原理

我们从 `关注人数`， `最近更新日期`， `贡献人数` 三个维度給备选方案进行排名，最后再按照各维度的排名进行汇总从而出总排名，排名越靠前表现越好，说明更值得选择。

为什么是这三个维度？ 我是这么认为的：

- 关注人数可以反应项目火热程度。
- 最后更新日期可以反应项目的维护情况，是否有被作者弃坑的风险。
- 贡献人数说明项目开源社区参与度。

好了，下面我们来看一个实际的例子。

### 举个例子

Go 语言里面选择哪个包管理工具更好？

我们的备选方案有:

- golang/dep
- Masterminds/glide
- tools/godep
- kardianos/govendor
- pote/gpm

当我们使用 which-better 来评测的结果为：

```
$ which-better -r golang/dep,Masterminds/glide,tools/godep,kardianos/govendor,pote/gpm

golang/dep: 3
Masterminds/glide: 8
tools/godep: 9
kardianos/govendor: 11
pote/gpm: 15
```

可以看到 `golang/dep` 在三个维度都是第一名，优势明显，所以在这个例子中我会毫不犹豫的选择 `golang/dep` 作为包管理工具。

### 写在最后

如果你对这个项目感兴趣的话，可以访问地址 https://github.com/songjiayang/which-better 查看具体的安装步骤。

这么好用的工具你还等什么？ just enjoy it.