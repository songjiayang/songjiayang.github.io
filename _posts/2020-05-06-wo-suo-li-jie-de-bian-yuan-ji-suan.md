---
title: "国内各云厂商边缘计算布局"
date: 2020-05-06 16:31:56+0800
thumbnail-img: /images/edge-compute/edge-computing-01.png
tags: [边缘计算]
---

## 背景

随着物联网、5G 等概念的提出，边缘计算变得越来越火热，它主要是利用边缘就近的算力、网络、存储等资源，在产生数据的边缘侧实现就近计算，从而降低延迟，提升性能，满足物联网、5G 时代对大连接、大带宽、低延迟的需求。

边缘计算主要分为云、边、端三个层级：

![edge-computing-01.png](/images/edge-compute/edge-computing-01.png)

> 图片来源：https://help.aliyun.com/document_detail/63839.html?spm=a2c4g.11186623.6.545.4e14545e8O8TXg

边缘计算主要解决云边端协同的问题，协同问题的整体内容如下：

![edge-computing-02.png](/images/edge-compute/edge-computing-02.png)

> 图片来源： 边缘计算与云计算协同白皮书(2018年) | 06

对此国内各家云厂商纷纷发力，针对云边端协同相继推出不少产品。这些产品主要分为两类：

- 第一类是侧重于资源出售的边缘计算。
- 第二类是侧重于物联网（端）的边缘计算。

本篇文章主要讨论侧重于资源的边缘计算的现状，后面会再写一篇偏向侧重于物联网（端）的边缘计算的介绍。

## 侧重于资源的边缘计算特点

侧重于资源的边缘计算的服务主要由云厂商提供边缘计算的资源，资源以云主机或边缘容器为主要的载体，一站式提供融合、开放、联动、弹性的分布式算力资源。

用户无需采购和自建节点，按需付费，扩容非常方便，这种边缘计算的资源主要集中分布在各地级市，其网络包括各运营商，能够做到全网覆盖。

## 典型应用场景：

- 实时音视频（互动直播 、在线教育）
- 客户端虚拟化（远程桌面、云游戏、VR）
- IOT 算力增强（边缘AI）
- 视频监控（就近观看，存储，大连接）
- 探针业务（节点地域覆盖广，网络类型丰富）

## 国内厂商现状：

<style>
table th {
    text-align: left;
}

table th:nth-child(1){
    width: 100px;
}

table th:nth-child(2), th:nth-child(3), th:nth-child(4){
    width: 80px;
}
</style>

|厂商        | 服务名称    |算力类型| 上线时间|主打应用场景|
| -------- | -----   | ---- |----|---- |---- |
| 阿里云  | [ENS](https://www.aliyun.com/product/ens) |虚拟机| 2018年 |互动直播、在线教育、视频监控、内容分发、业务监测|
| 腾讯云  | [ECM](https://cloud.tencent.com/product/ecm) | 虚拟机  |2020年|实时音视频、云游戏 、边缘 AI|
| 京东智联云 | [JEES](https://www.jdcloud.com/cn/products/jd-cloud-equal-edge-service) | 容器  |2019年|探针业务、视频监控、云游戏|
| 华为云    | 暂无    |   -    | -|- |
| 百度云    | 暂无    |   -    | -|- |
| 金山云    | [KENC](https://www.ksyun.com/post/product/KENC.html)  | 容器  | 2019 年| 客户端虚拟化、直播优化、IoT算力增强 |
| UCloud   | [UEDN](https://www.ucloud.cn/site/product/uodn.html)  | 虚拟机 | 2020 年 | 视频直播、监控平台、网络爬虫、AI-Service|

## 总结

通过上述表格可以看到，伴随腾讯云和 UCloud 相关服务的推出，各云厂商基本完成了侧重于资源的边缘计算的布局，说明针对边缘计算的资源托管和出售的业务的需求是真实存在的。

各厂商服务的功能基本相似，主推的应用场景也大体相似，但算力类型却有不同，京东智联云和金山云所推的是容器服务，而其它厂商都为虚拟机。

现在很难说明虚拟机和容器孰优孰劣， 虚拟机有更好的隔离性，更传统的使用体验，容器有更好的性能，更强的弹性伸缩能力。目前边缘计算主要的业务场景为无状态的业务，两种方式都能够很好的满足，但未来会如何演变，让我们一起保持关注。

下一讲，我们将主要介绍国内云厂商在侧重于物联网（端）的边缘计算的布局情况，敬请期待。

------

参考资料：

- https://www.aliyun.com/product/ens
- https://cloud.tencent.com/product/ecm
- https://www.jdcloud.com/cn/products/jd-cloud-equal-edge-service
- https://www.ksyun.com/post/product/KENC.html
- https://www.ucloud.cn/site/product/uodn.html


