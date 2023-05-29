---
title: "个人常用 web 框架吞吐率对比"
date: 2016-05-27 22:31:00 +0800
tags: [效率]
---


### 目的

测试个人经常使用的不同 App server（主要用来做 API）的吞吐率。

### 测试主机

- 系统： MacBook Air (11-inch, Early 2015)
- CPU: 1.6 GHz Intel Core i5 x4 (4 核)
- 内存： 4 GB 1600 MHz DDR3

### 参数对比

##### 1. Golang Gin 框架（协程模式）
- Go 1.6
- Gin v1.0rc1

##### 2. Node Koa 框架 (多进程的异步IO模式)
- Node v6.1.0
- Koa 1.2.0

##### 3. Ruby Sinatra 框架 (多进程多线程模式)
- Ruby 2.3.1p112
- Sinatra 1.4.7
- Puma 3.4.0

### 测试方法
- 使用 wrk 测试工具
- 压力参数参考 http://www.littlelines.com/blog/2014/07/08/elixir-vs-ruby-showdown-phoenix-vs-rails/
- 分别测试 `/users` (有 DB 查询) 和 `/static` （无 DB 查询）

### 数据准备

采用 mongodb 作为测试，并选用了各自最为流行的 mongo 驱动。

进入 mongo console 执行以下命令：

```
use ruby2go2node2elixir

db.users.insert([
  {"_id": ObjectId(), "username": "Josh Williams", "created_at": new Date(), "updated_at": new Date()},
  {"_id": ObjectId(), "username": "Josh Williams", "created_at": new Date(), "updated_at": new Date()},
  {"_id": ObjectId(), "username": "Josh Williams", "created_at": new Date(), "updated_at": new Date()},
  {"_id": ObjectId(), "username": "Josh Williams", "created_at": new Date(), "updated_at": new Date()},
  {"_id": ObjectId(), "username": "Josh Williams", "created_at": new Date(), "updated_at": new Date()}
])
```

### 启动服务：

所有命令都是在项目根目录前提下执行：

##### Gin (golang)（绑定在3000端口）
```
cd go
source env.sh
make
```

##### Koa (nodejs)（绑定在3001端口）
```
cd nodejs
npm install
npm start
```

##### Sinatra (ruby)（绑定在3002端口）
```
cd ruby
bundle
puma -C puma.rb
```

### Wrk 本地测试结果

#### Gin (golang)

```
wrk -t4 -c100 -d30S --timeout 2000 "http://127.0.0.1:3000/static"
Running 30s test @ http://127.0.0.1:3000/static
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    14.63ms   59.10ms 779.91ms   96.68%
    Req/Sec     5.42k     1.37k   20.49k    86.36%
  634806 requests in 30.10s, 167.09MB read
Requests/sec:  21092.78
Transfer/sec:      5.55MB

wrk -t4 -c100 -d30S --timeout 2000 "http://127.0.0.1:3000/users"
Running 30s test @ http://127.0.0.1:3000/users
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    17.63ms    5.65ms 125.03ms   80.56%
    Req/Sec     1.43k   317.13     2.19k    59.50%
  171245 requests in 30.03s, 143.06MB read
Requests/sec:   5702.07
Transfer/sec:      4.76MB
```

#### Koa (nodejs)

```
#### 多进程模式

wrk -t4 -c100 -d30S --timeout 2000 "http://127.0.0.1:3001/static"
Running 30s test @ http://127.0.0.1:3001/static
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    10.77ms    8.97ms 199.54ms   94.74%
    Req/Sec     2.56k   374.25     4.50k    87.25%
  305517 requests in 30.03s, 84.20MB read
Requests/sec:  10172.22
Transfer/sec:      2.80MB

wrk -t4 -c100 -d30S --timeout 2000 "http://127.0.0.1:3001/users"
Running 30s test @ http://127.0.0.1:3001/users
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    44.25ms   18.20ms 460.36ms   85.57%
    Req/Sec   576.93     83.73     0.85k    76.90%
  68859 requests in 30.01s, 56.08MB read
Requests/sec:   2294.72
Transfer/sec:      1.87MB

##### 单进程模式

wrk -t4 -c100 -d30S --timeout 2000 "http://127.0.0.1:3001/static"
Running 30s test @ http://127.0.0.1:3001/static
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    21.21ms    1.45ms  47.63ms   93.30%
    Req/Sec     1.18k   103.09     1.88k    86.25%
  141520 requests in 30.06s, 39.00MB read
Requests/sec:   4708.48
Transfer/sec:      1.30MB

wrk -t4 -c100 -d30S --timeout 2000 "http://127.0.0.1:3001/users"
Running 30s test @ http://127.0.0.1:3001/users
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    73.37ms   10.07ms 212.29ms   87.93%
    Req/Sec   342.42     66.59   505.00     63.45%
  40923 requests in 30.08s, 33.33MB read
Requests/sec:   1360.40
Transfer/sec:      1.11MB

```


#### Sinatra (ruby)

```
wrk -t4 -c100 -d30S --timeout 2000 "http://127.0.0.1:3002/static"
Running 30s test @ http://127.0.0.1:3002/static
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    22.83ms   28.96ms 392.78ms   88.99%
    Req/Sec     1.23k   334.43     2.07k    60.92%
  147405 requests in 30.06s, 35.85MB read
Requests/sec:   4903.72
Transfer/sec:      1.19MB

wrk -t4 -c100 -d30S --timeout 2000 "http://127.0.0.1:3002/users"
Running 30s test @ http://127.0.0.1:3002/users
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   153.62ms  121.39ms   1.09s    73.08%
    Req/Sec   159.87     36.28   287.00     69.44%
  19111 requests in 30.05s, 16.51MB read
Requests/sec:    635.96
Transfer/sec:    562.70KB

```

###  吞吐率倍数关系

- 无 DB 查询:

```

  gin > koa(多进程) > koa(单进程) ~= sinatra   
     (2x)          (2x)
```

- 有 DB 查询:
```
  gin > koa(多进程) > koa(单进程) > sinatra   
     (2x)          (2x)         (2x)
```


### 结论：

1. 在 4 核 CPU 条件下， golang 并发大概是 Nodejs 的两倍，Ruby 8 倍。因为 golang 有更好的并发模型，所以核数越多，这个差距越大 。
2. 多进程的并发模型，代价其实挺大的。 比如 Node 多进程模式并发能力并没有随核数成倍增加，大致是其核数／2 的提升。
3. Ruby GIL 缘故，其并发能力最差不足为奇。但是现在主流的都是微服务，RESTful API，一些性能要求较高的 业务完全可以考虑其他性能更好的语言。在其他语言领域，有些 ruby 棘手的问题，或许根本就不存在。
