---
title: "Use Textfile Collector to make your node exporter more Powerful"
date: 2016-11-08 22:31:00 +0800
tags: [prometheus]
---

有时 node_exporter 不能完全满足机器信息的收集，相较于 fork 代码，使用官方推荐的 Textfile Collector 是一种更简便的方式。

### Textfile Collector 使用逻辑大致如下：  

1. 使用 `-collector.textfile.directory` 参数启动 node_exporter。
2. 将我们自己收集的数据，按照 prometheus 的 [文本格式](https://prometheus.io/docs/instrumenting/exposition_formats/#text-format-details) 存储到 textfile collector 目录下的 *.prom 文件中。
3. 每当 Prometheus server 到 node_exporter 拉数据的时候，node_exporter 会自动解析所有 *.prom 文件，将数据全部返回。


### Prometheus 文本格式的规则

- 使用 `#` 开头表示注视，将被忽略。
- 如果以 `# HELP` 开头，将视为 Metric 帮助信息。
- 如果以 `# TYPE` 开头，将视为 Metric 类型定义，如果没有定义类型，Metric 默认为 Gauge, 例如：

```
# Minimalistic line:
metric_without_timestamp_and_labels 12.47

# HELP http_requests_total The total number of HTTP requests.
# TYPE http_requests_total counter
http_requests_total{method="post",code="200"} 1027 1395066363000
http_requests_total{method="post",code="400"}    3 1395066363000
```

可以看到文本格式的数据都是静态的，它并不是一个可以执行的脚本，这意味着我们为了获动态的数据，需要定时执行一些数据收集脚本，并将数据转化为 Prometheus 识别的文本格式，最后同步到对应的 `*.pom` 文件即可。

在日常工作中，我们总结了一套比较好的模式，大致过程如下：

- 在 node_exporter 目录中新建一个 cron 目录，用于存放所有需要执行的脚本, 脚本按照执行的时间间隔分组，分别放到不同的目录，所以 cron 目录应该是这样：

```
cron
├── 120
│   └── collector1.sh
├── 300
│   └── collector2.sh
└── main.cron.py
```

- 每个脚本负责一个 Metric 数据收集，它只负责将收集的数据转化为对应的文本格式，然后输出到标准 IO。
- 有个主函数，主函数的输入为一组需要执行的脚本目录，他会依次执行收集脚本，然后将每个脚本输出的标准 IO，以 `.pom` 为后缀的文件，保存到配置的 Textfile Collector 目录。
- 使用 contab 定时执行主函数。
```
*/2 * * * * python ～cron/main.cron.py ~/cron/120
*/5 * * * * python ～cron/main.cron.py ~/cron/300
```

总结： 很方便使用 Textfile Collector 来扩展 node_exporter，收集一些自定义需求。 Textfile Collector 和 pushgateway 很相似，只不过 pushgateway 更多的是 service 级别数据收集，而 Textfile Collector 更多是 node_exporter 补充，收集机器信息。
