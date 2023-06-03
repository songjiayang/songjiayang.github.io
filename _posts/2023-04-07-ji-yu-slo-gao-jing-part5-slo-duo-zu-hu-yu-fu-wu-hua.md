---
title: "基于 SLO 告警（Part5）：SLO 多租户与服务化"
subtitle: 
date: 2023-04-07  22:10:38+0800
thumbnail-img: /images/slo/cortex/01.jpg
# cover-img: 
# share-img: 
tags: [slo]
---

在本系列第 3 和 4 篇文章中，我们讲解了如何使用开源项目 sloth 和 pyrra 进行 SLO 告警，它们能够满足大多数场景需求，一般使用方法有：

- 使用对应命令行与 GitOps 结合。
- 使用对应 Operator 与 K8S 结合。

但如果是一个多租户的平台产品（SaaS 服务），我们又该如何使用它们呢? 

此问题核心是如何构建一个支持多租户的 SLO 统一服务，主要功能包括：

- 租户级别的 Alert Windows 统一管理。
- 租户级别的 SLO 统一管理，Prometheus rules 生成（与特定 Alert Windows 绑定），以及与已有的 ruler 集成。

## 方案设计

社区兼容 Prometheus 告警并支持多租户的开源产品不少，例如 Cortex、Mimir、VM 等，这里我们主要以 Cortex/Mimir 为例。

以前文章我们讲过，Cortex/Mimir 自己的分布式 ruler 实现了多租户的 Prometheus rules 存储和评估，单个租户的 rules 以 namespace 和 group 进行管理，存储在对象存储的文件名如下：

```
rules
└── demo
    └── myservice
        └── group1
            ├── rule1.yaml
            └── rule2.yaml
```

相似思路，我们也可以使用对象存储来存储租户的 SLO 相关配置，整体方案如下

![01.jpg](/images/slo/cortex/01.jpg)

方案说明：

- 新增 cortex-gatway，用于统一的租户鉴权，支持 Basic 和 JWT Token。
- 新增 cortex-slo，用于租户的 SLO 配置统一管理。
- 使用 sloth 作为 Prometheus rules generator，并将生成的结果通过 Cortex API 存储到 Ruler。
- 扩展 cortex-tools，方便使用命令行进行租户的 SLO 管理。

这里我们使用 sloth 作为 generator 主要因为它相较于pyrra，其生成的 SLI 指标更直观，而且它支持自定义 Alert Windows。

最终存储在对象存储中的 SLO 相关文件路径如下:

```
slos
└── demo
    └── myservice.yaml
windows
└── demo
    └── google-30d.yaml
```

## 实际使用

本地使用 docker-compose 启动 cortex 集群（带有 ruler 和 alertmananger）、grafana、cortex-slo、cortex-gateway，当成功运行后，使用 cortex-tools 进行 SLO 管理。

使用命令的时候，我们可以通过环境变量 TOWER_AUTH_TOKEN 指定租户的 JWT Token，TOWER_ADDRESS 指定 Cortex gateway 地址。

### 租户 Alert Windows 管理

- 添加告警 windows

```
$ cortextool slos load-windows config/slos/windows/*.yaml

INFO[0000] 7d.yaml windows loaded                       
INFO[0000] google-30d.yaml windows loaded 
```

- 列表查询 windows

```
$ cortextool slos list-windows

INFO[0000] Windows:                                     
7d
google-30d
```

另外还能使用 get-windows 查询单个 windows 详细信息，delete-windows 删除某个 windows 。

### 租户 SLO 配置

- 批量加载配置

```
$ cortextool slos load ./config/slos/*.yml

INFO[0000] myservice.yml slos loaded with default alert windows 
```

在没有指定 windows 的情况，会使用系统默认的 windows （google-30d）进行 SLI 和告警规则生成, 此时我们打开 Grafana 页面，可以看到刚生成的 rules，使用了30d的时间窗口。

![02.jpg](/images/slo/cortex/02.jpg)

当我们修改了 SLO 配置或者 alert windows 只需要重新 load 配置即可：

```
$ cortextool slos load ./config/slos/*.yml --windows=7d

INFO[0000] myservice.yml slos loaded with 7d alert windows
```

此时刷新 Grafana rules 页面，可以看到已经使用 7d 的告警窗口。

![03.jpg](/images/slo/cortex/03.jpg)

- 删除 SLO

```
$ cortextool slos delete myservice

INFO[0000] myservice slos deleted 
```

当执行成功后，再到 Grafana 页面查看，你会发现 myservice 相关的告警规则已经不见了。

## 总结

本篇文章我们主要讲解如何依靠开源 sloth 的 generator，构建 cortex-slo 统一服务，实现对 Prometheus SLO 的自动化管理，并扩展了 cortex-tools 命令行工具进行快捷操作。

如果您恰好有用 Cortex，而又想实现 SLO 的统一服务，那敬请期待后续我们的 cortex-slo 的项目开源。

至此，基于 SLO 告警系列文章已经完结，希望对您有所帮助。