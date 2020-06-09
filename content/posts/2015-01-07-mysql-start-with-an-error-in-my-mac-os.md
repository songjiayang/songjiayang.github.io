---
title: "mysql start with an error in my mac os"
date: 2015-01-07 22:31:00 +0800
tags: [mysql]
---

元旦放假归来，打开电脑，启动 mysql.server 得到一如下错误提示：

> . ERROR! The server quit without updating PID file (/usr/local/var/mysql/dev.pid).

查看错误日志，发现有这么一句提示：

```
2015-01-04 11:53:45 1694 [Warning] Setting lower_case_table_names=2 because file system for /usr/local/var/mysql/ is case insensitive
2015-01-04 11:53:45 1694 [Note] Plugin 'FEDERATED' is disabled.
/usr/local/Cellar/mysql/5.6.21/bin/mysqld: Can't find file: './mysql/plugin.frm' (errno: 13 - Permission denied)
```

已经隐隐约约感觉到是权限问题了，那修改此目录的 owner 为当前用户试试：

```
sudo chown -R dev /usr/local/var/mysql/
```
重新启动 mysql, 错误提示不见了，问题解决。
