---
title: 两个查看 Linux 端口占用情况的命令
date: 2014-01-21 22:42:26 +0800
tags: [linux] 
---

#### `netstat` 和 `lsof`是两个很好的查看服务器上端口占用情况的命令，简单使用如下：

```
=> netstat -tupln
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:27017         0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:11211         0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:28017         0.0.0.0:*               LISTEN      -


=> lsof -i
COMMAND     PID        USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
chrome     3081 songjiayang   78u  IPv4 285465      0t0  TCP sjy-computer.lan:44393->ea-in-f113.1e100.net:https (ESTABLISHED)
chrome     3081 songjiayang  101u  IPv4 282282      0t0  TCP sjy-computer.lan:44344->ea-in-f113.1e100.net:https (ESTABLISHED)
```
