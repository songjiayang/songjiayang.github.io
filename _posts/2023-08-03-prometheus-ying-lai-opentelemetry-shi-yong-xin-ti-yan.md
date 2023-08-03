---
title: "Prometheus 迎来 OpenTelemetry—使用新体验"
subtitle: 
date: 2023-08-03 22:21:19+0800
# thumbnail-img: 
cover-img: /images/prometheus/otlp/1.jpg
# share-img: 
tags: [prometheus]
---

OpenTelemetry 经过几年发展，其 SDK 已然成为数据埋点、采集、上报的默认选择，在去年 【Mimir 原生支持 OTLP，离大一统又近了一步】一文中我们讲到，Mimir 从 v2.3.0 开始原生支持 OTLP，它底层存储引擎用的是 Prometheus TSDB，所以这部分改动很容易反馈给上游，今天（2023/07/28）相关代码终于合并到了 main 分支，相信很快就会发布正式版本。

下面我们就来快速测试并体验该功能。

## 启动 Prometheus 测试容器

因为 Prometheus 还没发布对应功能的正式版本，所以使用最新主分支的镜像 prom/prometheus:main来测试，对应 docker 命令如下：

```
docker run --name prometheus-otlp \
    -d -p 9090:9090 \
    prom/prometheus:main \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/prometheus \
    --web.console.libraries=/usr/share/prometheus/console_libraries \
    --web.console.templates=/usr/share/prometheus/consoles \
    --enable-feature=otlp-write-receiver
```

注意 otlp-write-receiver 属于实验性功能，需要通过 --enable-feature 开启。

成功启动后，可以通过 HTTP 接口（api/v1/otlp/v1/metrics）进行验证，此时执行 `curl -X POST http://localhost:9090/api/v1/otlp/v1/metrics` 应该响应 400 而不是 404。

## 测试代码

使用 OpenTelemetry Go SDK（v0.39.0）编写测试代码，主要分为以下几步。

### Step1: 新建 otlpmetrichttp exporter

```go
ep, _ := otlpmetrichttp.New(
    context.Background(),
    otlpmetrichttp.WithEndpoint("localhost:9090"),
    otlpmetrichttp.WithURLPath("/api/v1/otlp/v1/metrics"),
    otlpmetrichttp.WithInsecure(),
  )
```

使用 Prometheus（localhost:9090） 作为 otlpmetrichttp 接收地址，路径为 /api/v1/otlp/v1/metrics， 在测试的时候，我们发现 Prometheus 的 otlp 仅支持 HTTP协议，GRPC 暂不支持。

### Step2: 新建 resource 和 provider

```go
ctx := context.Background()
res, _ := resource.New(ctx,
    resource.WithAttributes(
        semconv.ServiceName("service1"),
        semconv.ServiceNamespace("staging"),
        semconv.ServiceVersion("v0.1"),
        semconv.ServiceInstanceID("instance1"),
    ),
)

provider := metric.NewMeterProvider(
    metric.WithResource(res),
    metric.WithReader(
        metric.NewPeriodicReader(ep, metric.WithInterval(15*time.Second)),
    ),
)
```

通过 resource 标注了该测试程序对应的服务名、命令空间、版本和实例 ID等信息，这个比较重要，因为最终会对应 Prometheus 的 job 和 instance 标签。

### Step3: 使用 provider 新建指标并进行采样

```go
meter := provider.Meter("http",
    api.WithInstrumentationVersion("v0.0.1"),
)

httpDurationsHistogram, _ := meter.Float64Histogram(
    "http_durations_histogram_seconds",
    api.WithDescription("Http latency distributions."),
)

opt := api.WithAttributes(
    attribute.Key("method").String(http.MethodGet),
    attribute.Key("status").String("200"),
)

elapsed := float64(rand.Intn(100))
httpDurationsHistogram.Record(context.Background(), elapsed, opt)
```

## 运行结果

测试程序的指标数据可以通过 Prometheus 的看板进行查询。

![otlp-console.jpg](/images/prometheus/otlp/1.jpg)

通过该图可知，OTLP 协议解析和转化后，生成 Prometheus 指标的标签对应关系如下：

resource 标签：

- job -> ServiceNamespace/ServiceName
- instance -> ServiceInstanceID
- ServiceVersion 没有使用

meter 标签：分别对应 Prometheus metrics 标签（全部保留）。

## 简单总结

当我们开启 otlp-write-receiver 试验功能后， Prometheus 具备了 otlpmetrichttp 协议数据的接收能力，非常方便实现从客户端程序或者 OTel Collector 直接将数据通过 OTLP 格式发送给 Prometheus。

在测试 Prometheus 进行 OTLP 协议数据接收的时候，需要注意以下几点。

- 通过 --enable-feature=otlp-write-receiver 开启该功能。
- 目前仅支持 otlpmetrichttp 协议，API 路径为 /api/v1/otlp/v1/metrics。
- SDK 需要通过 resource 指定 ServiceName 和 ServiceInstanceID，进行多服务多实例指标的区分。

参考代码 [prometheus-examples/otlp-write-receiver](https://github.com/songjiayang/prometheus-examples/tree/main/otlp-write-receiver)。