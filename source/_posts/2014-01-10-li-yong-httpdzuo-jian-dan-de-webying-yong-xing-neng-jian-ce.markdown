---
layout: post
title: "利用httpd做简单的web应用性能检测"
date: 2014-01-10 23:37:05 +0800
comments: true
categories: [web, performace, tool, ubuntu] 
---


> 网站上线之前一般都会先检查下网站的抗压能力，比如并发和最大请求数等。那么我们可以使用httpd来做一些简单的测试。其中Httpd又简称ab.

<h4>1.ubuntu上Ab安装 </h4>
执行```sudo apt-get install httpd ```会得到诸如 "Package httpd is a virtual package provided by: "的提示，这是因为httpd是作为一个lib，会被给出的包所自动安装，所以随便选择一个提示的包就好，比如选择`apache2-mpm-worker 2.2.22-6ubuntu5.1`。
安装命令:
```
sudo apt-get install apache2-mpm-worker 2.2.22-6ubuntu5.1
```

<h4>2.Ab命令的使用 </h4>
使用ab命令进行测试的时候，主要用到的参数-c 和-n ; 其中-c 表示并发量（模拟同时多少人访问），-n表示一共发送的http请求数，所谓请求数量越大，数据相对越准确。


**以Ruby-China为例子**

测试语句：
 `ab -c 20 -n 2000 http://ruby-china.org/`  
执行结果:
```
Server Software:        nginx/1.4.2
Server Hostname:        ruby-china.org
Server Port:            80

Document Path:          /
Document Length:        33568 bytes

Concurrency Level:      20
Time taken for tests:   116.274 seconds
Complete requests:      2000
Failed requests:        1128
   (Connect: 0, Receive: 0, Length: 1128, Exceptions: 0)
Write errors:           0
Non-2xx responses:      22
Total transferred:      68131781 bytes
HTML transferred:       66399247 bytes
Requests per second:    17.20 [#/sec] (mean)
Time per request:       1162.744 [ms] (mean)
Time per request:       58.137 [ms] (mean, across all concurrent requests)
Transfer rate:          572.22 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       17  144  70.0    140    1183
Processing:    18 1017 422.2   1005    2925
Waiting:       18  302 163.9    261    1429
Total:         37 1160 417.0   1147    3041

```

通过分析结果可以知道，ruby-china 使用的是nginx web 服务器，大约每秒可以处理的请求是17个，每个请求大概耗时1162ms, 网络流量572Kb/每秒 .

