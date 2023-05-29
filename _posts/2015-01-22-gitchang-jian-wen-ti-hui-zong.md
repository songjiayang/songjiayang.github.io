---
title: "git 常见问题汇总"
date: 2015-01-22 22:31:00 +0800
tags: [效率, git]
---

#### 1. `git log` 常用命令：

```
下一条:  return
下一页:  space bar
前一页:  w
退出:  q
帮助:  h
```

#### 2. `git tag` 常用命令：

```
git tag -a v1.4 -m 'my version 1.4' #当前提交添加 tag
git tag -a v1.4 9fceb02  #给任意提交添加 tag
git tag -d v1.4 # 删除一个tag
git push origin :refs/tags/v1.4 # 删除远程仓库tag
git show v1.4
git push origin --tags
```

参考： http://git-scm.com/book/en/v2/Git-Basics-Tagging
