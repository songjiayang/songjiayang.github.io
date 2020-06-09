---
title: "CloudNativeCon 2017 Prometheus 报道"
date: 2017-08-18 22:02:05
tags: [prometheus]
---

最近两天 PromCon 2017 正如火如荼的进行着，具体演讲主题以及大会安排可以参考链接 http://t.cn/RCUGHoy，或者访问 Twitter 频道 #PromCon2017查看最新动态。

目前大会还未结束，暂时无法获取相关 slide, 但是我们可以回顾一下今年 3/29～ 3/30 的 CloudNativeCon 2017 大会，下面是我整理的关于 Prometheus 的分享主题和以及对应的视频，资料链接（需要翻墙）。

![pic1.png](/images/promconf2017/pic1.png)

主要介绍 Prometheus 的成长历史，如何从一个公司内部系统到多人贡献的优秀开源系统的华丽转变。
- video: http://t.cn/RCyH0mA
- slide: http://t.cn/RCyQluy

![pic2.png](/images/promconf2017/pic2.png)

主要介绍 Prometheus 的 Counter 数据模型，最佳实践。
- video: http://t.cn/RCynpyG
- slide: http://t.cn/RCyQluy

![pic3.png](/images/promconf2017/pic3.png)

主要介绍最新版本 AlertManager 的高可用模型, 再也不用担心告警的 HA 问题了。
- video: http://t.cn/RCyugqk
- slide: 缺失

![pic4.png](/images/promconf2017/pic4.png)

主要介绍 Cloud Native 生产环境中如何使用 Prometheus 监控告警，释放运维人力的案例。
- video: http://t.cn/RCUUOvt
- slide: 缺失

![pic5.png](/images/promconf2017/pic5.png)

详细介绍 AlertManager 的架构，各个组建作用，配置示例，并根据实践为 AlertManager 写了一个告警历史查询的插件。
- video: http://t.cn/RCUUESz
- slide: http://t.cn/RCUUkk6

![pic6.png](/images/promconf2017/pic6.png)

主要介绍 Grafana 配合 Prometheus 使用，以及指出不满足的地方，并使用扩展给出解决方案。
- video: http://t.cn/RCU4w4O
- slide: http://t.cn/RCU4GY0

![pic7.png](/images/promconf2017/pic7.png)

主要介绍 Prometheus 超长时间跨度的存储方案
- video: http://t.cn/RCU4ac1
- slide: http://t.cn/RCUvYu5

![pic8.png](/images/promconf2017/pic8.png)

主要介绍如何使用 mtail 和 snmp_exporter 对负载均衡设备进行监控，从而对系统的关键组件性能更加了解。
- video: http://t.cn/RCU40tg
- slide: 缺失

![pic9.png](/images/promconf2017/pic9.png)

主要介绍通过配置参数，对 Prometheus 进行调优，大大提高性能，实战性非常强。
- video: http://t.cn/RCU4Tdk
- slide: http://t.cn/RCUhDgd

![pic10.png](/images/promconf2017/pic10.png)

主要介绍如何使用 Prometheus 监控一个基于微服务架构，容器，容器编排的开源系统，如果你具有相似架构，那么此视频具有很强的参考价值。
- video: http://t.cn/RCU43Ty
- slide: http://t.cn/RCyFe4f

![pic11.png](/images/promconf2017/pic11.png)

这是一个非常有趣的话题，众所周知，Prometheus 主要用于后端系统的数据收集，但是它也可以用来收集浏览器的一些指标，演讲者，脑洞大开，让人耳目一新。
- video: http://t.cn/RCU4DPH
- slide: http://t.cn/RCUb20o

以上就是 CloudNativeCon 2017 中关于 Prometheus 的 11 场分享，其中有些 slide 暂时没有找到，后续会尝试联系作者提供下载链接（点击阅读原文，可以跳转视频地址，需要翻墙）。

在这 11 场分享中，我最感兴趣的话题是  **Configuring Prometheus for High Performance** 和 **Understanding and Extending Prometheus AlertManager**，收获颇多。

最后，预祝 PromCon 2017 取得圆满成功，早点放出视频，让我等大饱眼福。





