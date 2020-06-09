---
title: How to solve "Out Of Sort Memrory" problem of mysql
date: 2014-03-14 11:38:29 +0800
tags: [mysql]
---

This Morning, i meet a mysql error when did sphinx index rebuild, it is *sql_fetch_row: Out of sort memory, consider increasing server sort buffer size* .

As the error message told, this problem caused by the limit of mysql sort_memory config, so we just increase the size and reload mysql.server .


How to config mysql?

1. open mysql config named `my.cnf`, it's */usr/local/opt/mysql/my.cnf* in my computer.

2. go to *sort_buffer_size* line and update the value to a suitable numerical. if you do not set this value it will be 2M size (for my computer)

3. run `mysql.server restart` to restart mysql.

problem solved , so happy.
