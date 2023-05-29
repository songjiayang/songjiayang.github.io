---
title: "前后端分离简单玩法"
date: 2016-06-26 22:31:00 +0800
tags: [架构]
---

前后端分离原因，这里就不展开讨论了，大家可以去了解下相关技术文章。

需求背景：

1. 前后端分离。
2. 利用 cookies 记住会话。

具体实践：

#### 路由设计：

用户直接访问的页面，全部由前端框架来 routing, 所有 API 请求以 `/api` 开头, 并被 nginx 代理到后端应用里 。

为什么要这么设计？

因为我们需要利用 cookies 来记录会话，但是 cookies 有同域保护的问题（子域名也不行），
那么用户访问地址和 API 请求必须具有相同的域名。

API 的请求全部拥有统一的前缀，只是为了方便 nginx 作代理请求。

代码如下：

```
location /api {
  expires -1;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $http_host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Scheme $scheme;
  proxy_redirect off;
  proxy_pass http://localhost:9090; // Application location
  break;
}
```

#### 前端处理 404 问题：

默认前端只有 `index.html` 页面， 那么当用户访问 `example.com/index2.html` 的时候，
如果不做特殊处理，将得到一个默认的 404 页面。

但是很有可能，在经过前端 js routing 过后，发现 `index2.html` 页面是存在的。

所以判断页面存在与否，取决于前端代码 routing 匹配过后结果，显然无论用户访问怎样的路由，
我们都应该先返回 `index.html`，然后执行前端代码，根据访问的路由来确定展现页面。

这里我们可以偷个懒，直接使用 nginx 的 error_page, 将其指向 index 页面即：

```
    error_page 404 /index.html
```

#### 开发模式下，解决同域的问题：

由于开发也是前后端分离的，前端人员开发不依赖 nginx, 那么可以找个能够修改域的中间件来解决这问题，
例如我们使用了 `http-proxy-middleware` 。

结论：

想要拥有前后端分离的好处，又想使用 cookies 记录会话，不妨采用这种方式，只需简单修改 nginx 即可，侵入极小。
