---
layout: post
title: "为什么选择nginx服务器"
date: 2014-01-11 22:29:50 +0800
comments: true
category: archive
tags: [tool, nginx]
---

以前Apache一直是web静态服务器领域的老大哥，占有绝对的市场分额，但是这种局面随着nginx的出现而不断改变。

现在越来越多的大公司已经从apache迁移到了nginx，主要原因就是nginx的超高的性能，甚至远远超过apache.那么了解nginx的安装，部署基本成为了每一个web工程师的必修课(尤其是Rails工程师).

**1. 通过源码安装Nginx**

* 从[nginx官网](http://nginx.org/en/download.html)下载源代码 (选用最新的stable版本，假设下载到本地~/Download目录)
* `sudo mv ~/Download/nginx-1.4.4.tar.gz /var/local` 
* `cd /var/local/ ; sudo tar -xvf nginx-1.4.4.tar.gz `
* `sudo ./config`
* `sudo make`
* `sudo make install`
* `cd /usr/local/nginx/`  

好了，如果没有报错的情况下的化，那么nginx已经安装成功了，并且默认安装到了/user/local/nginx目录.在/user/local/nginx目录下执行`ls -l`你将看到如下输出:
{% highlight  ruby%}
drwx------ 2 nobody root 4096  1月 11 22:13 client_body_temp
drwxr-xr-x 2 root   root 4096  1月 11 22:22 conf  #nginx配置文件
drwx------ 2 nobody root 4096  1月 11 22:13 fastcgi_temp
drwxr-xr-x 2 root   root 4096  1月 11 22:12 html #nginx默认网页
drwxr-xr-x 2 root   root 4096  1月 11 22:22 logs
drwx------ 2 nobody root 4096  1月 11 22:13 proxy_temp
drwxr-xr-x 2 root   root 4096  1月 11 22:12 sbin #nginx命令文件
drwx------ 2 nobody root 4096  1月 11 22:13 scgi_temp
drwx------ 2 nobody root 4096  1月 11 22:13 uwsgi_temp
{% endhighlight %}

安装完成后，可以使用`sudo ./sbin/nginx` 启动nginx, 如果没有任何输入，那么表示启动成功，如果提示*(98: Address already in use)*之类的表示nginx启动的端口已经被占用,那么只需停掉该端口占用的进程或者修改nginx的监听的端口（默认80）就好。

Nginx的默认配置文件是`conf/nginx.conf`,如果想修改监听端口只需如下：
{%highlight ruby%}
    server {
        listen       81;  #将原来的80修改为任意未被占用的端口
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
    ...
    }

{%endhighlight%}



**2. Nginx和Apache服务器性能简单比较**

Nginx执行`ab -n 10000 http://localhost:81/index.html` 的结果是:
{% highlight ruby%}
Concurrency Level:      1
Time taken for tests:   2.239 seconds
Complete requests:      10000
Failed requests:        0
Write errors:           0
Total transferred:      8440000 bytes
HTML transferred:       6120000 bytes
Requests per second:    4466.03 [#/sec] (mean)
Time per request:       0.224 [ms] (mean)
Time per request:       0.224 [ms] (mean, across all concurrent requests)
Transfer rate:          3680.99 [Kbytes/sec] received

{% endhighlight %}

Apache执行`ab -n 10000 http://localhost/index.html` 的结果是:

{% highlight ruby%}
Concurrency Level:      1
Time taken for tests:   4.453 seconds
Complete requests:      10000
Failed requests:        0
Write errors:           0
Total transferred:      4550000 bytes
HTML transferred:       1770000 bytes
Requests per second:    2245.57 [#/sec] (mean)
Time per request:       0.445 [ms] (mean)
Time per request:       0.445 [ms] (mean, across all concurrent requests)
Transfer rate:          997.79 [Kbytes/sec] received

{% endhighlight%}

以上是两个服务器在没有任何调优的情况下的测试结果,通过数据不难发现，nginx的请求量/每秒基本比apache多了1/3,可见nginx不是一般的快,这就是为什么我使用nginx的原因。




