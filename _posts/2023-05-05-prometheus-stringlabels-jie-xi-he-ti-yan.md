---
title: "Prometheus stringlabels 解析和体验"
subtitle: 
date: 2023-05-05 23:11:24+0800
thumbnail-img: /images/prometheus/stringlabels/01.jpg
# cover-img: 
# share-img: 
tags: [prometheus]
---

Prometheus v2.44.0 开始，默认采用 stringlabels 存储 Labels，它能够降低 CPU 和内存（-20%）的使用，下面我们就来看看它是如何做到的？

## 先聊聊 slicelabels

我们都知道 Prometheus 的指标是由一系列标签对（Labels）组成的，例如：

```
request_count{job="web", route="query_range"}
```

在过去 Prometheus 版本中, Labels 是使用字符串数组（slicelabels）来存储，所有标签对按照 [key,value] 排序，然后 append 到同一 slice ，结构大致为：

![01.jpg](/images/prometheus/stringlabels/01.jpg)

这种存储方式虽然代码逻辑简单，但其内存利用率并不高，因为除了要存储 k/v 本身内容外，还增加了 slice 头（24 字节）和每个字符串头（16 字节）的额外开销。

例如一个包含 10 个键值对，所有 k/v 都是 10 个字节的指标，使用 slicelabels 需要的内存大致为 24+10*2*(16+10)=544 字节，但我们标签实际内容总共才 2*10*10=200 字节。

可见 slicelabels 并不是一种非常经济的存储方案，所以才有了 stringlabels。

## stringlabels 如何解决问题

stringlabels 放弃了 slice 存储多个字符串，而采用单一字符串来存储整个 Labels，结构大致为：

![02.jpg](/images/prometheus/stringlabels/02.jpg)

每个 k/v 字符串，先记录其长度（length prefix），然后再追加具体内容，最终统一编码到同一字符串，如果想查找/获取某些标签对，都需要从头解析获取。

这种编码方式内存开销为 string 固定 16 字节头和 每个k/v 的长度标记（varint）以及具体的 k/v内容，还是上面样例，使用 stringlabels 所占内存可以减小为 16+10*2(1+10)=236，不到 slicelabels 的50%。

相关核心代码都在 model/labels/labels_string.go 文件中。

## 测试使用

这里使用 avalanche 来构建测试 metrics，并分别用两个 不同版本的 Prometheus 抓取对应指标，过一段时间后，可以发现 v2.44.0 版本所占内存（默认开启 stringlabels）为 v2.43.0（非 stringlabels ）80% 不到。

![03.jpg](/images/prometheus/stringlabels/03.jpg)

相关配置如下：

- docker-compose.yaml

```
version: '3.4'
services:
  prometheus:
    image: prom/prometheus:v2.43.0
    user: root
    command: ["--config.file=/etc/prometheus/prometheus.yaml"]
    volumes:
      - ./prometheus.yaml:/etc/prometheus/prometheus.yaml
      - ./data/data-prometheus-1:/prometheus
    ports:
      - 9090:9090

  prometheus2:
    image: prom/prometheus:v2.44.0-rc.0
    user: root
    command: ["--config.file=/etc/prometheus/prometheus.yaml"]
    volumes:
      - ./prometheus.yaml:/etc/prometheus/prometheus.yaml
      - ./data/data-prometheus-2:/prometheus
    ports:
      - 9091:9090

  avalanche:
    image: prometheuscommunity/avalanche:main
    command: 
      - --metric-count=1000
    ports:
      - 9001:9001
```

- prometheus.yaml

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: avalanche
    static_configs:
      - targets: ['avalanche:9001']
  - job_name: prometheus
    static_configs:
      - targets: ['prometheus:9090', 'prometheus2:9090']
```

## 总结

Prometheus 的 stringlabels 带来了不错的内存节省，随着作为即将发布的Prometheus v2.44.0 的默选项，标志着它日趋稳定，大家可以体验起来了。