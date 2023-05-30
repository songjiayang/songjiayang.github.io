---
title: "通过反向代理解决非 SSL 资源引入的问题"
date: 2016-02-23 22:31:00 +0800
tags: [效率]
---

SSL 网站里出现非 SSL 资源会出现什么情况呢？

![invalid_ssl_link.png](/images/invalid_ssl_link.png)

看到了吗，浏览器会有警告，像这样 **this page includes other resources which are not secure**。

你或许认为这个属于第三方的资源，并非不安全。但是真实情况中，用户可不这么想，因为他们不懂技术。
他们开始担心隐私，数据安全; 接着开始抱怨，更有甚者停止使用你的网站了。

那有什么办法解决这个问题呢？答案就是 **“使用反向代理”**。

反向代理的过程大致是：

#### 将所有用户输入的资源链接，转化成走代理服务器的安全链接,例如：
```
<img src='http://example.com/images/foo.png' />
转化为
<img src='https://proxied-assets/example.com?original_src=http://example.com/images/foo.png'>
```

转化的核心是反向代理资源链接是安全的，并且可以得到原始链接（对应关系，可以自己设计）。

#### 代理服务器根据请求链接，得到原始链接，然后请求该原始链接，获得内容，然后将内容转发。

这个过程看起来简单，但是有一些注意地方：

1. 反向代理服务器，必须高性能，IO 异步最好，所有选择 NodeJS 较好。
2. 反向代理服务器，下载原始链接内容的时候，需要设置超时处理，以及不同返回状态，还有 http 和 https 不同协议的处理。
3. 代理服务器返回原始链接内容的时候，最好跟原始链接头信息相同，尤其是过期时间之类。
4. 可以在反向代理服务器和用户之间再添加一层 CDN, 用户直接从 CDN 下载资源，CDN 从代理服务器下载，这样做可以减少代理服务的 网络 IO 请求。


刚看了下 github issue 的图片处理，发现他们也是采用类似的思路，他们使用了 [camo](https://github.com/atmos/camo) 这个库。参考其实现, 我迅速的给 ruby-china 提了个 [pull request](https://github.com/ruby-china/ruby-china/pull/576) :smile:.

-----

参考链接：

- [https://github.com/blog/743-sidejack-prevention-phase-3-ssl-proxied-assets](https://github.com/blog/743-sidejack-prevention-phase-3-ssl-proxied-assets)
- [https://github.com/atmos/camo](https://github.com/atmos/camo)
