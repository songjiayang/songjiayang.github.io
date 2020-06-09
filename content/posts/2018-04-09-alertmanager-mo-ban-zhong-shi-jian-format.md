---
title: "如何格式化 Alertmanager 告警模版中的时间"
date: 2018-04-09 17:08:44+0800
tags: [prometheus]
---

AlertManger 告警模版中显示的时间使用的是 Go 默认的格式，形如：

![images/alerttemplate/alert01.png](/images/alerttemplate/alert01.png)

显然这个格式不太好看，我们更期望使用的形式为 `2018-10-07 12:00:00`，那我们该如何来格式化呢？

这里主要使用 Go 的 `time.Format` 方法，其在 `template` 中的写法如下：

```golang
.StartsAt.Format "2006-01-02 15:04:05"
```

所以我的 `wechat.tmpl` 修改后的结果为：

<script src="https://gist.github.com/songjiayang/354f2b4387880d18c10185288e1d8a47.js"></script>

最后的告警效果图：

![images/alerttemplate/alert02.png](/images/alerttemplate/alert02.jpg)
