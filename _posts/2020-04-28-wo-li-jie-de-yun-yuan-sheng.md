---
title: "我所理解的云原生"
date: 2020-04-28 16:15:17+0800
# cover-img: /images/cloudnative/cloudnative-06.png
thumbnail-img: /images/cloudnative/cloudnative-01.png
# share-img: /assets/img/path.jpg
tags: [云原生]
---

云原生对应的英文是做 Cloud Native, 属于最近几年比较火的技术，那到底什么是云原生，它和 CNCF、Kubernetes、Docker 又是什么关系？

今天我们就从云原生的定义，解决的核心问题，以及主要的开源产品等维度出发，一起来探究云原生的本质，希望能够对它有个宏观的认识。

## 什么是云原生？

> Cloud native technologies are used to develop applications built with services packaged in containers, deployed as microservices and managed on elastic infrastructure through agile DevOps processes and continuous delivery workflows.

— Writes Janakiram MSV, principal analyst at Janakiram & Associates and an adjunct faculty member at the International Institute of Information Technology.

根据这段话我们可以大概知道，云原生并不是一种具体的技术，而是一类思想的集合，它包含了微服务（MicroServices）、DevOps、持续交付，以及容器等一系列技术。

## 为什么需要云原生？

长期以来，我们一直在追求三个目标：

- Availability (可用性)
- Scale （可伸缩性）
- Performance （性能）

但现在时代变了，仅有这三个目标是不够的(或传统方式在当今很难满足这三个指标），究其原因，主要是`速度`，我们在不断的追求速度。

- 追求速度能够给我们带来好处：

![cloudnative-01.png](/images/cloudnative/cloudnative-01.png)

- 追求速度意味着改变频率的加快：

![cloudnative-02.png](/images/cloudnative/cloudnative-02.png)

- 改变频率加快意味着故障概率增加：

![cloudnative-03.png](/images/cloudnative/cloudnative-03.png)

- 节点的故障概率影响可用性：

![cloudnative-05.png](/images/cloudnative/cloudnative-05.png)

可以看到这是一个连锁反应，那是否可以做到在追求速度的情况下，依然保证可用性，减少故障率呢？

这就是云原生存在的价值，它总结了很多成功的经验，集大家的思想，让我们避免前人踩过的坑，让我们在追求速度的前提下，依然能够保证可用性和弹性伸缩的能力，真正做到又快又好。

![cloudnative-04.png](/images/cloudnative/cloudnative-04.png)

## 什么是云原生应用？

前面我们已经了解了什么是云原生以及为什么需要云原生，那什么是云原生应用就好理解，即按照云原生思想部署的应用就可以叫做云原生应用。

云原生应用的 10 个特征：

- 打包为轻量级容器。

- 使用最佳语言和框架开发。

- 设计为松耦合的微服务。

- 以API为中心进行交互和协作。

- 以无状态和有状态服务的清晰分离为架构基础。

- 与服务器和操作系统依赖关系隔离。

- 部署在自服务、弹性、云基础架构上。

- 通过敏捷 DevOps 流程进行管理。

- 自动化功能。

- 定义的、策略驱动的资源分配。

特征详细解释参考链接 https://thenewstack.io/10-key-attributes-of-cloud-native-applications 。

## 云原生和 CNCF

前面讲到云原生是一套思想，它的落地需要各种工具。

![cloudnative-06.png](/images/cloudnative/cloudnative-06.png)

当今是一个开源的时代，我们绝大多数应用都构建在开源项目的基础上，云原生领域也是如此，开源社区有大量云原生相关的项目，首当其冲的就是 Kubernets，当 Google
将 kubernets 捐赠给 CNCF（云原生计算基金会）的那一刻起，标志着 CNCF 的正式诞生。

CNCF 作为一个厂商中立的基金会，致力于 Github 上的快速成长的开源技术的推广，帮助云原生开源产品的孵化，让云原生应用得到推广和普及。


我们可以通过 CNCF 全景图（https://landscape.cncf.io） 了解当前云原生相关开源产品的具体情况。

![cloudnative-07.png](/images/cloudnative/cloudnative-07.png)

从这个图我们可以看到，云原生相关的产品也非常之多，例如我们常见的：

- Kubernetes：最流行的容器编排工具，它能够屏蔽操作的差异，让我们应用部署、伸缩变得更加容易。
- Prometheus：云原生领域最流行的基于指标的监控告警工具，很多云原生应用或者开源项目直接集成了 Prometheus，通过 exporter 暴露了其指标。
- Jaeger： 分布式调用链追踪系统，它用来监控复杂的微服务环境并对其进行故障排除。
- containerd：开源可信赖的容器运行时，类似轻量版的 Docker。
- Istio： 服务网格最火的开源产品，为微服务提供保护、连接和监控微服务的统一方法。


写到这里已是文末，再次回到文章开头提出的几个问题，什么是云原生，它和 CNCF、Kubernetes、Docker 又是什么关系？我想此时大家心中已有答案。

希望通过本文能够帮助大家揭开云原生的神秘面纱，对其有个整体认识，希望你在云原生的实践中一帆风顺。

------

参考链接：

- [https://thenewstack.io/category/cloud-native](https://thenewstack.io/category/cloud-native)
- [https://thenewstack.io/10-key-attributes-of-cloud-native-applications](https://thenewstack.io/10-key-attributes-of-cloud-native-applications)
- [https://www.slideshare.net/AmazonWebServices/dmg206](https://www.slideshare.net/AmazonWebServices/dmg206)
- [https://www.cncf.io/wp-content/uploads/2017/11/What-is-Cloud-Native-CNCF-Webinar-23-Feb-2017-1.pdf](https://www.cncf.io/wp-content/uploads/2017/11/What-is-Cloud-Native-CNCF-Webinar-23-Feb-2017-1.pdf)