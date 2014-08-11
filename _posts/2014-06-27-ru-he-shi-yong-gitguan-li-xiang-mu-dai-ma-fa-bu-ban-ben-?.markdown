---
layout: post
title: "如何使用git管理项目代码发布版本"
date: 2014-06-27 10:49:22 +0800
comments: true
tags: [git, tag]
---

这里我们主要使用git的tag命令；tag命令的详情查看可以使用 `git tag —help`.  

以下是我经常用到的几条命令：

1.1 列出项目中所有的tags

```

git tag
```

1.2 给项目添加一个tag, 一般情况下tag都是以v开头命名的，例如v1.0.1

```
git tag vTAGNAME
```

1.3 给指定某一个commit 打上tag，一般用于tag修改和遗漏的tag

```
git tag vTAGNAME COMMIT-HASH
```
1.4 根据tag名称删除tag

```
git tag -d vTAGNAME
```

1.5 推送所有的tags到远程仓库

```
git push origin —tags
```

