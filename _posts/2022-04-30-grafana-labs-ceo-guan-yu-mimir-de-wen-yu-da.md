---
title: "Grafana Labs CEO 关于 Mimir 的问与答"
subtitle: 
date: 2022-04-30  22:58:54+0800
thumbnail-img: /images/mimir/qa/01.jpg
# cover-img: 
# share-img: 
tags: [grafana, mimir]
---

Mimir 刚开源的时候，我们第一时间写了一篇关于它的不权威解读，在经过几个月的功能体验和源码阅读后，有一些心得想和大家分享：

- Mimir 尽可能的将各个模块无状态化，使用 Gossip 协议做集群数据同步，避免了使用中心式的 KV 存储，从而使部署更简单，架构更清晰。
- 在 Prometheus/Thanos/Cortex 上做了大量针对性修改，尤其是大规模写入和查询，这块可以参考后面我们的 “ Mimir 是如何支持单DB 10亿指标” 系列文章。
- 对多租户管理比较友好，每个租户都可以单独配置限制，租户有详细的监控指标，您可以做到针对某个租户进行限流和封禁等操作。
- Mimir 团队确实在不断回馈社区，他们将修改不断反馈给上游，大家可以关注下 grafana/mimir-prometheus 这个仓库，很多新的特性，在不久将来应该都会合并到 Prometheus 中，例如（chunk 异步落盘、split&merge 压缩、乱序样本数据写入 ）。

如今又重看当时 Grafana CEO 对 Mimir 开源的问答，更是感慨颇多，故将部分内容摘抄下来。

## 部分问答内容

以下部分文字摘自翻译 https://grafana.com/blog/2022/03/30/qa-with-our-ceo-about-grafana-mimir

### 1、什么是 Grafana Mimir?

这是我们针对 metrics 的下一代开源可扩展的时序数据库。它包含了 Cortex 代码和部分商业版的能力。代码路径在https://github.com/grafana/mimir 这是基于AGPLv3 协议下的开源产品。

在 Mimir 中，我们在原有 Cortex 的基础上，采用一种水平扩展的分片压缩架构，实现了基数不受限制的能力。这个已经在超过1亿指标的生产环境中得到验证。Mimir 借助水平扩展的分片查询引擎，提供了强悍的快速查询能力。我们发布了开箱即用的 Grafana 面板、告警和大量示例，让 Mimir 的使用更便捷。与此同时，大量的自动化运维工具：TSDB 检查，bucket 分析，流量分析等也可以帮你快速上手。

Mimir 不只适用于 Prometheus 类型的数据格式，我们很快会支持 OpenTelemetry, Graphite, Influx, 和 Datadog 等类型的metrics数据写入。此外，我们还删除了一些废弃代码，简化了很多配置。

### 2、为什么要做改变

时间会检验一切，我们在商业版中的很多能力，希望能开放出去，得到市场更多的反馈。很多企业使用了我们的开源版本，但是却没有给我们一些正向反馈。所以，原有的 Apache 2.0 将无法同时兼顾开源版和商业版的发展，所以我们使用了AGPLv3。

### 3、 Mimir是 Cortex 的复制吗？为什么不把这些特性放到Cortex 中？
是的，Mimir 来自于 Cortex 的复制。

在 Grafana，我们相信开源的力量，这是我们的 DNA。我们还相信，您可以通过拥有可持续发展的业务来获得最好的 OSS，该业务可以聘请世界上最好的开发人员来推动项目向前发展。这是我们在 Grafana Labs 的使命。

Cortex 是 CNCF 管理的基于 Apache2.0 协议的开源项目。其中大部分贡献也来自于 Grafana 的人员。Cortex 被一些世界上最大的云提供商和 ISV 使用，他们能够以较低的成本提供 Cortex，因为他们在开发项目上投入的资金并不相同。

这使我们不得不选择从保持改进 Cortex 到闭源这条路。我们认为我们已经找到了一种使其开放并建立可持续业务的方法，而不是关闭代码，我们觉得 AGPLv3 许可证和 CLA 的结合实现了这一点。

.....

## 最后

最后看一下 Cortex 和 Mimir 的对比：

![01.jpg](/images/mimir/qa/01.jpg)

此时，您是否和我们一样，越发相信 Mimir 是 Prometheus 生态的第一选择。