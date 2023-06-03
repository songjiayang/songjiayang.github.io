---
title: "Prometheus Native Histogram 实现原理及应用"
subtitle: 
date: 2022-11-09 21:05:04+0800
thumbnail-img: /images/histogram/native/02.jpg
# cover-img: 
# share-img: 
tags: [prometheus]
---

今天(2022/11/08) Prometheus 发布了 v2.40.0 版本，其中增加了 Native Histogram 的支持，这个功能的改动还是挺大的，从设计到开发完成历时两年多，下面我们就来一起看看它的实现原理和应用。

## 当前 Prometheus Histogram 的问题

在正式开始 Native Histogram 探究之前，先来看看目前 Prometheus Histogram 存在的问题。

当前 Prometheus Histogram 格式如下：

![01.jpg](/images/histogram/native/01.jpg)

可以看到默认 Histogram 需要预先定义桶分布，默认值为（.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10），不同桶的值是累积的，每个桶都是单独指标，而且需要额外增加 'le="+Inf"','_count','_sum' 三个指标。 

由于这样的设计，导致如下问题:

- 查询精度，由于需要业务提前定义桶分布，这对开发来说有一定的心智负担，因为提前固化桶的分布，所以无法根据实际的请求值动态增加桶的数量，从而导致最终结果在某些范围（超过最大桶）出现较大偏差。
- 存储膨胀：单个Histogram 指标的不同桶以及 'le="+Inf"', '_count', '_sum' 三个额外指标都属于不同时间线，需要单独存储与查询。
- 不同桶的分布无法计算：同名指标不同label之间聚合，需要严格进行桶对齐（按照 le 相同的指标进行聚合），例如：histogram_quantile(__quantile__, sum by(le) (rate(__histogram_metric__[5m])))

## Native Histogram 解决办法

针对以上问题，Prometheus 社区引入了 Native Histogram 方案（以前叫做 high sparse histograms 及高稀疏直方图），下面我们就来看看 Native Histogram 的实现原理。

注意，下面图片转载至 codesome 的分享（https://docs.google.com/presentation/d/1161g3FTRzZ68fgOAwVpCfszSi62FNraTxPWhOYEFehQ）

### 固定桶的边界

在原始 Histogram 中，我们可以自定义任意桶分布，但在 Native Histogram 使用了固定桶边界的方案。

用户只需要关心其解析度（相邻两个桶宽度比）即可计算出采用的是哪种模式，故得出使用的是哪种固定桶边界方案。目前支持的桶边界固定分布方案一共有 12 种，它们的倍数因子分别为 2、4、16、256 以及 2的(1/2)次方~2的（1/256）次方，具体分布如下图：

![02.jpg](/images/histogram/native/02.jpg)

![03.jpg](/images/histogram/native/03.jpg)

这个因子对应 golang_client SDK 中的 NativeHistogramBucketFactor 参数，其实我们平时在使用的时候，并不需要关注底层具体使用的是哪种模式，只需要关心相邻两个桶后面一个比前面一个增加多少即可，比如增加 10%，那么该值为 1.1 即可， SDK 会按照最接近这个比例匹配对应的固定桶分布。

### 数据编码

可以看到新的 Native Histogram 完全放弃了原始的编码格式，不再使用 le="+Inf", _count, _sum，而采用了固定桶分布的方式，所有我们可以给桶进行标号，只要我们知道其解析度就可以完全还原出所有编号对应的桶区间。

![04.jpg](/images/histogram/native/04.jpg)

然后基于该桶编号进行 Native Histogram 数据编码：

![05.jpg](/images/histogram/native/05.jpg)

这里我们首先记录 metadata 信息，包含解析度、sum、count 等信息，当解析度较小时，固定桶会比较多，可能出现大量桶无值的情况，这里引入 Spans 概念，表示有值的连续桶区间，这样只需记录有值的桶即可。

现在单个 Native Histogram 只需一个时间线进行存储，故最终对应的 chunk 编码结构如下：

![06.jpg](/images/histogram/native/06.jpg)

这种编码带来的好处是 Prometheus TSDB Block Index 的减小，参考社区数据，在全是 Histogram 的情况下，索引文件达到 94% 减少。

![07.jpg](/images/histogram/native/07.jpg)

## 实战练习

我们使用 prometheus/client_golang 构建 Demo App，定义一个 NativeHistogram 指标:

```
httpDurationsHistogram := prometheus.NewHistogramVec(prometheus.HistogramOpts{
  Name:                         "http_request_durations",
  Help:                         "HTTP latency distributions.",
  Buckets:                      prometheus.DefBuckets,
  NativeHistogramZeroThreshold: 0.05,
  NativeHistogramBucketFactor:  factor,
}, []string{"service", "code"})
```

参数说明：

- NativeHistogramZeroThreshold 用于 zero bucket 定义，及所有小于 0.05 的值都统计到该桶。
- NativeHistogramBucketFactor 定义 NativeHistogram 桶的解析度，这里设置 1.5 表示期望在 1～2 之间再划分一下，用到的固定桶区间为 (1,1.414213562373095], (1.414213562373095,2]
- 针对 Native Histogram 通常我们只需要关心这两个参数，另外还有 NativeHistogramMaxBucketNumber、NativeHistogramMinResetDuration、NativeHistogramMaxZeroThreshold 这三个参数做更细粒度的配置。

这里显示设置 Buckets 为默认配置，主要是为了兼容低版本的 Prometheus 抓取。

### 运行体验

测试代码已经提交到 play-with-prometheus-native 仓库，大家可以使用如下命令运行测试:

```
git clone https://github.com/grafanafans/play-with-prometheus-native.git
cd play-with-prometheus-native
docker-compose up -d
```

该命令会启动 native-demo app， 监听 8080 端口，然后再启动两个 Prometheus 实例进行指标抓取，一个支持 Native Histogram（监听9090），一个不支持（监听9091）。

具体 docker-compose.yaml 如下：

```
prometheus:
  image: prom/prometheus:v2.40.0
  command:
    - --config.file=/etc/prometheus/prometheus.yml
    - --log.level=error
    - --storage.tsdb.path=/prometheus
    - --web.console.libraries=/usr/share/prometheus/console_libraries
    - --web.console.templates=/usr/share/prometheus/consoles
    - --enable-feature=native-histograms
    - --enable-feature=exemplar-storage
  volumes:
    - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
    - prometheus-1-data:/prometheus
  ports:
    - 9090:9090
prometheus2:
  image: prom/prometheus:v2.40.0
  command:
    - --config.file=/etc/prometheus/prometheus.yml
    - --log.level=error
    - --storage.tsdb.path=/prometheus
    - --web.console.libraries=/usr/share/prometheus/console_libraries
    - --web.console.templates=/usr/share/prometheus/consoles
    - --enable-feature=exemplar-storage
  volumes:
    - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
    - prometheus-2-data:/prometheus
  ports:
    - 9091:9090
native-demo:
  image: songjiayang/native-histogram-demo:v0.1.0
  ports:
    - 8080:8080
  command: 
    - --native-factor=1.5
    - --metrics-count=10
```

我们可以修改 native-demo app 的运行参数，进行不同 NativeHistogramBucketFactor 测试。

当程序启动后，即可访问 http://localhost:9090 查看 Native Histogram 的数据：

![08.jpg](/images/histogram/native/08.jpg)

我们做了大量测试，目前版本对 Histogram 兼容性做的还可以，包括低版本 Prometheus 抓取带有 Native Histogram 的数据，新版本 Prometheus 抓取老版本 Exporter 数据，而且对 exempalr 数据也能支持。

测试过程中我们也发现一些问题，比如同时做大量 series 做 p95 计算的时候，Native Histogram 比原始 Histogram 耗时久一些，大约是其 2x-5x 倍的计算耗时，详情参考 issues/11548。

## 总结

从设计到研发历时两年多，Prometheus 终于在 v2.40.0 发布了 Native Histogram 的第一个正式版本，它通过固定桶边界的方式，简单的配置即可在保证桶解析度的情况下，实现时间线存储的空间的极大优化。

新版本对老版本兼容支持做的也比较好，能够向后兼容，从个人试用结果看 Native Histogram 并非没有缺陷，比如大量时间线（1000+）同时计算 p99 耗时远远大于默认的 Histogram，而且目前它只支持 ProtoBuf 格式数据抓取，这给大家 调试 Exporter 带来了一定困扰。

但瑕不掩瑜，这个算是一个非常大的功能，感兴趣的小伙伴可以尝试起来。