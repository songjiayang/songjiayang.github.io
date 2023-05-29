---
title: "Linux 进程管理方法总结"
date: 2015-08-12 22:31:00 +0800
tags: [linux]
---

作为一名后端工程师，难免会与服务器打交道，懂些常见基本运维技能将有助于你快速排查问题，提高工作效率；其中进程管理属于最基本的入门技能。

写在前面： 所有例子跑在 ubuntu 14.0.4 上，其他 linux 版本也基本相同。

### top 命令

```
top
```

```
top - 14:18:10 up 298 days,  1:59,  2 users,  load average: 0.21, 0.23, 0.22
Tasks:  77 total,   1 running,  76 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni,  0.0 id, 99.3 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   1017972 total,   954880 used,    63092 free,      152 buffers
KiB Swap:  1048572 total,   655128 used,   393444 free.    13588 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 3743 deploy    20   0  869224 184396    748 S  0.0 18.1 316:32.87 ruby
30840 deploy    20   0  836504 163364    808 S  0.0 16.0   0:10.59 ruby
28860 mysql     20   0 1474952 143452      0 S  0.0 14.1   0:07.11 mysqld
31049 deploy    20   0  767176 122260      0 S  0.0 12.0   0:05.98 ruby
10764 deploy    20   0  749244  28852      0 S  0.0  2.8   3:19.23 ruby
 1391 deploy    20   0   25144   6352    492 S  0.0  0.6   0:00.21 bash
  600 syslog    20   0  256360   2744      0 S  0.0  0.3   0:45.98 rsyslogd  
  ...
```


如上图所示，使用 top 命令，不仅可以看到所有正在运行的用户进程，还有系统资源使用统计信息，比较简单直观。

提示： 使用 `shift + m` 可以根据内存使用率排序。

我经常使用这个命令查看内存使用情况，寻找占用内存较大的进程。

### ps 命令

```
ps aux
```

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  33660  1120 ?        Ss    2014   0:09 /sbin/init
root         2  0.0  0.0      0     0 ?        S     2014   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S     2014  46:47 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S     2014   0:01 [kworker/u:0]
root         6  0.0  0.0      0     0 ?        S     2014   0:00 [migration/0]
....
```

aux 选项可以列出所有用户进程（除当前 terminal 进程）。

如果只是列出所有进程并不好用，毕竟进程太多，此时我们可以结合 grep 命令作关键词匹配，比如：

```
ps aux | grep mysql

deploy    1567  0.0  0.0  10460   916 pts/2    S+   14:06   0:00 grep --color=auto mysql
mysql    28860  0.0 14.0 1474952 143448 ?      Ssl  11:20   0:06 /usr/sbin/mysqld

```

我们还可以用 `ps PID` 显示某特定进程信息:

```
ps 28860

PID TTY      STAT   TIME COMMAND
28860 ?        Ssl    0:06 /usr/sbin/mysqld
```

### kill 命令

当我们发现某些进程僵死，或者想强制结束某个进程的时候，往往会使用 kill 命令。

最基本命令格式是： `kill PID`

kill 命令有很多选项，使用 kill -l 可以查看：

```

 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2

```

其实，平时我用的比较多的只有第 9 选项 `SIGKILL`。

最后在提一下，`kill PID` 和 `kill -9 PID` 的区别。

首先 kill PID 命令不会直接删除进程，而是向该进程发送一个关闭信息，这样好处是进程可以自己控制结束动作，在结束之前可以做一些额外操作。
但是对于有些应用程序，没有相应删除的实现，那么你就只有使用 －9 选项，强制结束进程。

### 总结

进程管理大致思路就是设法找到相应进程，查询进程运行状况，删除或重启进程。其中会结合使用 top, ps, kill 等命令。
