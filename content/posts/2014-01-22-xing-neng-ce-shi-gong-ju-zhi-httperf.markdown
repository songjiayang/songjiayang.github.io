---
title: 性能测试工具之 httperf
date: 2014-01-22 19:56:06 +0800
tags: [linux] 
---

1. 下载[httperf](https://httperf.googlecode.com/files/httperf-0.9.0.tar.gz)

2. 配置环境 `./config`

3. 编译 `make`

4. 安装 `make install`

5. httperf的基本使用

```
==> httperf  --hog  --server=www.ruby-china.org  --wsess=100,100,0.1  --burst-length=10  --rate=5  --timeout=5


httperf --hog --timeout=5 --client=0/1 --server=www.ruby-china.org --port=80 --uri=/ --rate=5 --send-buffer=4096 --recv-buffer=16384 --wsess=100,100,0.100 --burst-length=10
httperf: warning: open file limit > FD_SETSIZE; limiting max. # of open files to FD_SETSIZE
Maximum connect burst length: 1

Total: connections 100 requests 10000 replies 10000 test-duration 23.630 s

Connection rate: 4.2 conn/s (236.3 ms/conn, <=21 concurrent connections)
Connection time [ms]: min 3829.2 avg 3977.6 max 4149.9 median 3991.5 stddev 105.9
Connection time [ms]: connect 74.3
Connection length [replies/conn]: 100.000

Request rate: 423.2 req/s (2.4 ms/req)
Request size [B]: 71.0

Reply rate [replies/s]: min 304.4 avg 450.8 max 505.6 stddev 97.7 (4 samples)
Reply time [ms]: response 90.4 transfer 0.0
Reply size [B]: header 195.0 content 184.0 footer 0.0 (total 379.0)
Reply status: 1xx=0 2xx=0 3xx=10000 4xx=0 5xx=0

CPU time [s]: user 2.28 system 21.27 (user 9.7% system 90.0% total 99.7%)
Net I/O: 186.4 KB/s (1.5*10^6 bps)

Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
Errors: fd-unavail 0 addrunavail 0 ftab-full 0 other 0

Session rate [sess/s]: min 1.20 avg 4.23 max 5.20 stddev 1.91 (100/100)
Session: avg 1.00 connections/session
Session lifetime [s]: 4.0
Session failtime [s]: 0.0
Session length histogram: 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
```  

更多使用说明可以通过命令` man httperf` 来查看.
