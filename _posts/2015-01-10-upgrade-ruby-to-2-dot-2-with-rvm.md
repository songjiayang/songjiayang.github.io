---
title: "upgrade ruby to 2.2 with rvm"
date: 2015-01-10 22:31:00 +0800
tags: [ruby]
---

rvm 升级命令

```
rvm get head
rvm install 2.2
```

NOTES:  
1. 执行 rvm get head 的时候，需要 pgp 添加签名， 类似代码 gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3  
2. 执行 rvm install 2.2, 如果遇到有关 gmp lib版本冲突的问题，请先执行 brew upgrade gmp 升级 gmp.
