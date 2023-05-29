---
title: "如何在禁用爬虫的网站上使用七牛镜像"
date: 2014-11-21 22:31:00 +0800
tags: [效率]
---

测试服务器上由于某种原因需要禁用所有爬虫，完全禁用的代码很简单，
只需要在public下面的 robots.txt 文件中添加如下代码即可。

```
# See http://www.robotstxt.org/wc/norobots.html for documentation on how to use the robots.txt file
#
# To ban all spiders from the entire site uncomment the next two lines:
User-Agent: *
Disallow: /
```

上述代码虽然简单，但是确实有点太粗暴，因为它也会禁止七牛的爬虫（该网站使用了七牛的镜像服务）。

为了继续使用七牛的服务，需要做一些额外操作。

1. 修改七牛镜像源为一个以 cdn开头的二级域名, 例如 cdn.eradai.com。
2. 在public下面新建一个 `robots.allow.txt` 的文件，内容为空。
3. 修改 nginx 配置,在 server 里面添加如下代码：

```
# HTTP
server {
  ......

  location /robots.txt {
    if ($host = 'cdn.eradai.com') {
      rewrite ^/robots\.txt /robots.allow.txt last;
    }
  }
  .....
}
```

经过以上三步，就可简单解决七牛爬虫被禁止的问题，原理是利用 nginx 的重定向。
将一个特殊域名请求重定向到 robots.allow.txt,
而 robots.allow.txt，开放了所有爬虫协议。
