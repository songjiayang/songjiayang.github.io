---
title: "Prometheus with mysqld_exporter"
date: 2016-10-21 22:31:00 +0800
tags: [prometheus]
---

![mysqld_exporter](/images/mysqld_exporter.png)

一.  创建一个 MySQL 用户，并拥有适当的权限

```
CREATE USER 'mysqld_exporter'@'localhost' IDENTIFIED BY 'password';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysqld_exporter'@'localhost'
  WITH MAX_USER_CONNECTIONS 3;
```

二. 下载和启动 mysqld_exporter

```
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.9.0/mysqld_exporter-0.9.0.linux-amd64.tar.gz
tar -xzf mysqld_exporter-0.9.0.linux-amd64.tar.gz
export DATA_SOURCE_NAME='mysqld_exporter:password@unix(/var/run/mysqld/mysqld.sock)/'
./mysqld_exporter
```

如果安装成功，访问 http://localhost:9104/metrics，您将看到 mysqld_exporter 所有 metrics

三. 修改 prometheus 配置

```
# 修改 prometheus.yml, 添加如下配置
scrape_configs:
 - job_name: 'mysqld'
   static_configs:
    - targets:
      - localhost:9104
```

重启 Prometheus，访问 http://localhost:9090/graph，然后查询 `mysql_up`, 你将看到 mysqld 的运行状态。
