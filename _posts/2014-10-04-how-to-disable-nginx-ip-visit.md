---
title: "How to disable nginx's ip visit"
date: 2014-10-04 22:31:00 +0800
tags: [nginx]
---

有时我们需要用户必须使用域名访问主机(及禁止IP访问)，只需简单修改下nginx配置即可：

```
server {
  listen 80 default;
  return 404; # or 500
}
```

上面代码意思是nginx监听ip访问的80端口，然后直接返回404/500; 其中404和500会对应默认的html页面。
