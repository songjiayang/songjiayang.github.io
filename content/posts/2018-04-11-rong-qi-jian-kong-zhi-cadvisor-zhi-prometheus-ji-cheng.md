---
title: "cadvisor 与 Prometheus 集成"
date: 2018-04-11 11:44:56+0800
tags: [prometheus, 云原生]
---

在上一篇文章 [容器监控之 cadvisor （一）](http://www.songjiayang.com/posts/jrong-qi-jian-kong-zhi-cadvisor) 已经介绍了如何通过 cadvisor 来收集容器的运行状态信息，那这篇文章将具体讲解如何与 Prometheus 集成并通过 Prometheus 查看收集的数据。

主要内容：

- cadvisor 与 Prometheus 集成
- 在 Prometheus 中查看容器的 CPU，内存，网络流量等信息


### cadvisor 与 Prometheus 集成

Step1: 修改 Prometheus 配置信息，添加 cadvisor 访问地址：

```
#prometheus.yml

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['127.0.0.1:9100']

  - job_name: 'container'
    static_configs:
      - targets: ['127.0.0.1:8080']  # 本地 cadvisor 访问地址
```

Step2: 重新加载配置，访问 `http://localhost:9090/targets` 你将看到新加的 `cadvisor` 已经生效。

![/images/cadvisor/prometheus01.png](/images/cadvisor/prometheus01.png)

Step3: 此时访问 Prometheus 的 graph 页面 `http://localhost:9090/graph`，搜索 `container` 你将看到容器相关数据。

![/images/cadvisor/prometheus02.png](/images/cadvisor/prometheus02.png)

### Prometheus 中查看容器的 CPU，内存，网络流量等数据

CPU 使用率查询：

```
sum by (name) (rate(container_cpu_usage_seconds_total{image!=""}[1m])) / scalar(count(node_cpu{mode="user"})) * 100
```

![/images/cadvisor/prometheus03.png](/images/cadvisor/prometheus03.png)

内存使用量：

```
sum by (name)(container_memory_usage_bytes{image!=""})
```

![/images/cadvisor/prometheus04.png](/images/cadvisor/prometheus04.png)

网络入口流量

```
sum by (name) (rate(container_network_receive_bytes_total{image!=""}[1m]))
```

![/images/cadvisor/prometheus06.png](/images/cadvisor/prometheus06.png)

网络出口流量

```
sum by (name) (rate(container_network_transmit_bytes_total{image!=""}[1m]))
```

![/images/cadvisor/prometheus05.png](/images/cadvisor/prometheus05.png)

磁盘使用量：

```
sum by (name) (container_fs_usage_bytes{image!=""})
```

![/images/cadvisor/prometheus07.png](/images/cadvisor/prometheus07.png)
