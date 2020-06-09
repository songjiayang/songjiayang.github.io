---
title: Linux 下磁盘性能简单检测
date: 2014-01-28 20:30:48 +0800
tags: [linux] 
---

1.使用dd命令简单测试磁盘I/O

```
dd if=/dev/zero of=testfile bs=1M count=512 conv=fdatasync
```
2.使用hdparm命令

```
sudo hdparm -t /dev/sda
```
