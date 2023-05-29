---
title: "使用 blackbox exporter 实现域名证书过期监控"
subtitle: 自动监控多个网站 TLS 证书过期，并及时告警，防止线上事故。
date: 2020-12-02 22:01:29+0800
cover-img: /images/prometheus/blackbox/tls/01.jpg
thumbnail-img: /images/prometheus/blackbox/tls/02.jpg
tags: [prometheus]
---

个人网站我一般使用 [Let's Encrypt](https://letsencrypt.org/) 的免费 CA 证书，当网站一多管理这些证书就显得比较麻烦，所以我非常希望有一个工具，能够将这些域名服务状态都列出来，包括证书过期时间、访问延迟、以及请求状态码等。

恰好上周看到 Grafana Labs 的一篇博客 [How we eliminated service outages from ‘certificate expired’ by setting up alerts with Grafana and Prometheus](https://grafana.com/blog/2020/11/25/how-we-eliminated-service-outages-from-certificate-expired-by-setting-up-alerts-with-grafana-and-prometheus/) 中讲到了使用 Grafana 的 表格视图来展示域名证书监控信息，觉得非常受用，惊呼这不就是我长期想要的东西吗?

![prometheus-tls-01.jpg](/images/prometheus/blackbox/tls/01.jpg)

<center>Grafana 监控面板效果图</center>

迫不及待，我立马在本地搭建了一套测试环境体验了一下，下面是整个测试过程。

## 本地测试环境搭建

我是在 Mac 上使用 Docker 搭建的测试环境，网络拓扑如下：

![prometheus-tls-02.jpg](/images/prometheus/blackbox/tls/02.jpg)

因为服务之间要相互访问，为了防止重启 container 导致IP地址变化，故创建了一个叫做 `mynetwork` 网段为 `172.18.0.0/16` 的自定义网络。创建容器的时候可以通过 `--ip` 手动分配静态 IP，各容器分配的 IP 地址如上图。

### 启动 nginx 容器

启动 nginx 之前我们需要创建测试域名的证书和配置文件，本次测试涉及三个域名（www.test01~3.com）。

#### 创建域名证书

使用 `genrsa.sh` 脚本批量创建自签名证书，脚本内容如下：

```
#!/bin/bash

#1、该脚本支持自动生成私有证书
#   ，用$1,$2....表示，第1,2...个参数，分表表示域名和过期时间

if [ "$#" -ne 2 ]; then
    echo "genrsa.sh www.test01.com 60"
    exit 1
fi

readonly DOMAIN=$1
readonly EXPIRATION=$2

openssl genrsa -out $DOMAIN.key 2048

openssl req \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=Tester/OU=Tester Software/CN=$DOMAIN/emailAddress=test@prometheus.show" \
    -new \
    -key $DOMAIN.key \
    -out $DOMAIN.csr


openssl x509 \
    -req \
    -days $EXPIRATION \
    -in $DOMAIN.csr \
    -signkey $DOMAIN.key \
    -out $DOMAIN.crt
```

运行效果：

![prometheus-tls-03.jpg](/images/prometheus/blackbox/tls/03.jpg)

#### 创建 nginx 配置文件

```

# touch nginx/conf.d/www.test01.com.conf
server {
    listen 443 ssl;
    server_name         www.test01.com;
    ssl_certificate     ssl/www.test01.com.crt;
    ssl_certificate_key ssl/www.test01.com.key;

    location / {
      return 200;
    }
}

# touch nginx/conf.d/www.test02.com.conf
server {
    listen 443 ssl;
    server_name         www.test02.com;
    ssl_certificate     ssl/www.test02.com.crt;
    ssl_certificate_key ssl/www.test02.com.key;

    location / {
      return 403;
    }
}

# touch nginx/conf.d/www.test03.com.conf
server {
    listen 443 ssl;
    server_name         www.test03.com;
    ssl_certificate     ssl/www.test03.com.crt;
    ssl_certificate_key ssl/www.test03.com.key;

    location / {
      return 200;
    }
}
```

- 创建了三个域名所需的证书和配置文件。
- 网站 www.test02.com 返回 403 状态码，其它返回 200。

执行容器启动命令

```
docker run -d --name nginx \
    --net mynetwork --ip 172.18.0.2 \
    -v `pwd`/nginx/ssl:/etc/nginx/ssl:ro \
    -v `pwd`/nginx/conf.d:/etc/nginx/conf.d \
    nginx
```

到此为止用于测试的 nginx 容器就启动起来了，它包了 www.test01~3.com 三个测试域名，每个域名都有自己的证书，每个证书过期时间不同。

### 启动 blackbox exporter

用于测试的网站已经部署完成，接下来部署 blackbox exporter。

#### 创建 blackbox.yml

```
modules:
  http_2xx:
    prober: http
    http:
      preferred_ip_protocol: "ip4"
      tls_config:
        insecure_skip_verify: true
```

- 这里使用 blackbox exporter 的 http 探针。
- 当执行 http 探测的时候，指定使用 ipv4 协议，因为是自签名证书，故配置 `insecure_skip_verify: true`。


#### 执行启动命令

```
docker run -d -p 9115:9115 \
    --name blackbox_exporter \
    --net mynetwork --ip 172.18.0.3 \
    --add-host www.test01.com:172.18.0.2 \
    --add-host www.test02.com:172.18.0.2 \
    --add-host www.test03.com:172.18.0.2 \
    -v `pwd`:/config \
    prom/blackbox-exporter:v0.18.0 --config.file=/config/blackbox.yml
```

要探测 www.test01~3.com 这三个域名，所以需要通过 `--add-host` 手动指定其解析的 IP 为 nginx 容器地址。


### 启动 Prometheus

当 blackbox exporter 部署完成后，我们需要部署 Prometheus 来抓取和存储数据。

更新 prometheus.yml 配置文件:

```
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://www.test01.com
        - https://www.test02.com
        - https://www.test03.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox01.com:9115
```

- 通过静态 static_configs 指定需要探测地址为 www.test01~03.com。
- 通过 relabel_configs 将实际的 metrics 获取地址重定向到 blackbox exporter 的地址，即我们配置的 172.18.0.3，这里使用了 `blackbox01.com` 域名来代替。

执行容器启动命令

```
docker run -d -p 9090:9090 \
    --name prometheus \
    --net mynetwork --ip 172.18.0.4 \
    --add-host blackbox01.com:172.18.0.3 \
    -v `pwd`/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus:v2.23.0
```

### 启动并配置 Grafana

经过上面的操作，已经能够在 Prometheus 的 graph 页面查看到 probe_ 开头的相关指标，我们可以更进一步，使用 Grafana 来展示。

#### 启动 Grafana 容器

```
docker run -d -p 3000:3000 \
    --name=grafana \
    --net mynetwork --ip 172.18.0.5 \
    --add-host prometheus01.com:172.18.0.4 \
    grafana/grafana
```

这里将 promtheus 地址映射为 `prometheus01.com` ，这样在 grafana 可以直接使用该域名来创建数据源。

#### 创建 Grafana 监控面板

使用 admin/admin 初始密码登录 http://localhost:3000 ，并使用 `prometheus01.com:9090` 创建默认 Promethues 数据源。

导入 ID 为 `13230` 的模板：

![prometheus-tls-04.jpg](/images/prometheus/blackbox/tls/04.jpg)

导入结果：

![prometheus-tls-05.jpg](/images/prometheus/blackbox/tls/05.jpg)

可以看到我们用于测试的这三个网站的探测信息都展示到 Grafana 图表了，展示内容包括我们想要的证书过期时间、状态码、连接各阶段耗时。

## 告警配置

除了可以通过 Grafana 面实时查看域名探测结果，我们可以配置 Prometheus rules 并结合 Alertmanager 实现告警监控。

告警配置如：

```
- name: ssl_expiry
  rules:
  - alert: Ssl Cert Will Expire in 7 days
    expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 7
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "域名证书即将过期 (instance {{ $labels.instance }})"
      description: "域名证书 7 天后过期 \n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

可能需要监控的指标：

- `expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 7` 实现域名过期检测，比如这里过期时间小于 7 天告警。
- `expr: probe_http_status_code != 200` 实现状态码不为 200的告警。
- `expr: sum(probe_http_duration_seconds) by (instance) > 1` 实现总耗时大于 1 秒的告警。

## 总结

今天我们在本地通过 Nginx 容器搭建了三个过期时间为 3d、60d 、365d 的测试域名，然后通过容器启动 Blackbox exporter 和 Prometheus 来探测和收集这三个域名的 metrics，最后通过 Grafana 的 13230 模板统一展示这个三个域名的探测结果。

通过测试可以看到，使用Blackbox Exporter 和 Prometheus 实现域名证书的探测和监控是非常方便的，而且 Grafana 的显示效果非常好，值得大家试试。