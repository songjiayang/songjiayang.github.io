---
title: 代码行统计之 git 命令
date: 2014-01-24 21:23:49 +0800
tags: [git] 
---

年底下，突然想看下今年项目提交代码的行数，于是在网上找到了如下命令:

```
git log --author="<USERNAME>" --pretty=tformat: --numstat | gawk '{ add += $1 ; subs += $2 ; loc += $1 - $2 } END { printf "added lines: %s removed lines : %s total lines: %s\n",add,subs,loc }'
```

具体使用只需要将<USERNAME>替换成某某一个git comment user，如：

```
git log --author="songjiayang" --pretty=tformat: --numstat | gawk '{ add += $1 ; subs += $2 ; loc += $1 - $2 } END { printf "added lines: %s removed lines : %s total lines: %s\n",add,subs,loc }'

added lines: 164821 removed lines : 287743 total lines: -122922

```
