---
title: "基于SLO告警（Part 4）：开源项目 pyrra 使用"
subtitle: 
date: 2023-02-09  21:09:58+0800
thumbnail-img: /images/slo/pyrra/01.jpg
# cover-img: 
# share-img: 
tags: [slo]
---

系列文章第3篇我们讲到如何使用 sloth 进行 Prometheus 规则的自动生成和 SLO 统一观测，今天我们再来看另外一个类似的开源项目 -- pyrra(https://github.com/pyrra-dev/pyrra)。

## Pyrra 简介

Pyrra 大部分功能和 sloth 类似，它提供了 4 个子命令，分别为：

- generate：它是最基础的命令，通过它实现规则文件手动生成。
- filesystem: 它在 generate 命令上，通过对特定目录监听的方式，实现规则文件自动生成。
- kubernetes: 它通过启动的 K8s Controller 实现对 ServiceLevelObjective 资源监听，生成 PrometheusRule 对象，实现与 Prometheus Operator 无缝对接。
- api: 它提供一个开箱即用的 UI，支持 SLO 列表和详情信息查询，适用于未部署 Grafana 的场景（当然 
pyrra 也提供了 Grafana 模板）。

### Pyrra SLO 配置格式

相较于 sloth，pyrra 只提供一种 SLO 格式，及自定义 K8s 资源 ServiceLevelObjective，内容大致为：

```
apiVersion: pyrra.dev/v1alpha1
kind: ServiceLevelObjective
metadata:
  name: pyrra-api-errors
  namespace: monitoring
  labels:
    prometheus: k8s
    role: alert-rules
    pyrra.dev/team: operations # Any labels prefixed with 'pyrra.dev/' will be propagated as Prometheus labels, while stripping the prefix.
spec:
  target: "99"
  window: 2w
  description: Pyrra's API requests and response errors over time grouped by route.
  indicator:
    ratio:
      errors:
        metric: http_requests_total{job="pyrra",code=~"5.."}
      total:
        metric: http_requests_total{job="pyrra"}
      grouping:
        - route
```

我们主要关注以下字段：

- metadata.name: 表示该 SLO 的名称。
- metadata.labels: 表示该 SLO 的标签，其中 pyrra.dev/ 开头的标签会去除前缀后，添加到最终生成的告警规则中。
- spec.target: 表示 SLO 目标值。
- spec.window: 表示 SLO 计算周期。
- spec.indicator: 表示 SLO 计算指标，支持 ratio 和 latency 两种类型。

pyrra 生成的 Prometheus 规则类型主要有：

- increase: 包含 SLO 周期总请求数和指标不存在的告警。
- metric_name:burnrate5m: 属于 record 记录，主要记录各时间窗口每秒的错误率。
- ErrorBudgetBurn: alert 告警，支持多窗口多燃烧率。
- generic：主要包含 SLO 的一些元信息，例如配置的 SLO 目标值，时间窗口大小等。

### Pyrra 与 Sloth 对比

优势：

1. 提供 filesystem 模式，实现对 SLO 目录文件变化，自动触发 generate 命令。
2. 提供 api 模式，一个开箱即用的统一看板。

不足：

1. 不支持 OpenSLO 格式。
2. 不支持时间窗口内更详细配置，例如短/长期窗口具体的告警燃烧率。
3. 不支持 Sloth 类似的 SLI 插件，对于通用服务（中间件）需要手动编写 SLIs。

实际使用中，如果我们没有太多窗口配置定制化，而且有 SLO 状态页面的需求（不方便使用 Grafana），pyrra 是个不错的选择。

## 实战练习

### 示例流程

我们还是以 MyService 为例，收集其指标并通过手动改变请求错误率的方式，对其 SLO 进行观测。

整个流程如下图：

![01.jpg](/images/slo/pyrra/01.jpg)

- 启动 MyService，能够通过接口进行错误率设置。
- 通过 pyrra filesystem 模式，自动生成 MyService SLO 对应的 Prometheus 规则。
- 启动 Prometheus，加载生成的规则文件，并收集 MyService 指标。
- 启动 Grafana 和 pyrra api，查看 SLO 列表和详情页面。

### 程序运行及效果

示例程序已提交到 https://github.com/grafanafans/play-with-pyrra 仓库，欢迎查看。

#### 启动程序

```
git clone https://github.com/grafanafans/play-with-pyrra.git
docker-compose up -d
```

当程序启动后，你将看到5个容器，它们分别为：

- Prometheus: http://localhost:9090
- Pyrra API: http://localhost:9099
- Grafana: http://localhost:3000
- MyService: http://localhost:8080
- Pyrra Filesyste

设置 MyService 错误率:

```
curl http://localhost:8080/errrate?value=0.005
```

当设置错误率为 0.5%（SLO 0.1% 的5倍），通过 Pyrra API 和 Grafana 观测到的 SLO 详情信息如下。

- Pyrra API 页面

![02.jpg](/images/slo/pyrra/02.jpg)

- Grafana 页面

![03.jpg](/images/slo/pyrra/03.jpg)

## 总结

可以看到 pyrra 和 sloth 做的事情类似，它也提供命令行和 K8s Controller 方式实现 Prometheus 规则的自动生成。相较于 sloth，它在服务化方面做的更多一些，不仅提供了一个开箱即用的 SLO 页面还支持 filesystem 模式对 SLO 目录进行监听。

总的来说，如果您有 SLO 状态页面的需求（不方便使用 Grafana），pyrra 是个不错的选择。
