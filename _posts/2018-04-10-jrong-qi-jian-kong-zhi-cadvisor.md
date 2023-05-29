---
title: "容器监控之 cadvisor"
date: 2018-04-10 17:49:39+0800
tags: [prometheus, 云原生]
---

想必大家在生产环境中已大量使用到了容器，那对于容器的监控（CPU, 内存，网络请求）是如何处理的呢？

### docker stats 对 cadvisor

众所周知 `dokcer stats` 可以查看运行的 Docker 镜像的运行状态，例如：

```
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT    MEM %               NET I/O             BLOCK I/O           PIDS
a25dd77a5237        cadvisor            0.91%               14.8MiB / 1.952GiB   0.74%               749kB / 11.5MB      18.9MB / 0B         11
```

这种方式比较原始，因为你无法通过 http 的方式来获取数据，而且没有界面，数据可视化还需要做大量的工作。

由于 `dokcer stats` 有这些问题，所以 `cadvisor` 诞生了。`cadvisor` 不仅可以搜集一台机器上所有运行的容器信息还提供基础查询界面和 http 接口，方便 Prometheus 进行数据抓取。

正是因为 `cadvisor` 与 Prometheus 的完美结合，所以它成为了容器监控的第一选择。

### cadvisor 的安装

Step1: 使用 `docker pull` 下载最新版本的 `cadvisor`

```
$ docker pull google/cadvisor:latest
```

Step2: 使用 `docker images` 查看下载的版本

```
$ docker images  

google/cadvisor      latest              75f88e3ec333        4 months ago        62.2MB
```

Step3: 使用 `docker run` 启动

```
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```

当启动成功后，使用 `docker ps` 你会看到

```
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                    NAMES
742a464fa631        google/cadvisor:latest   "/usr/bin/cadvisor -…"   1 second ago        Up 1 second         0.0.0.0:8080->8080/tcp   cadvisor
```

Step4: 访问 `http://localhost:8080` 你将看到：

![/images/cadvisor/cadvisor-01.png](/images/cadvisor/cadvisor-01.png)


这说明 `cadvisor` 已运行成功。

### cadvisor 深入了解

Tips1: 访问 `http://localhost:8080/docker` 可以查看到所有运行的 dokcer 镜像：

![/images/cadvisor/cadvisor-02.png](/images/cadvisor/cadvisor-02.png)

Tips2: 选择任意一个镜像，你将看到其运行状态的详细信息：

![/images/cadvisor/cadvisor-03.png](/images/cadvisor/cadvisor-03.png)

![/images/cadvisor/cadvisor-04.png](/images/cadvisor/cadvisor-04.png)

Tips3: 访问 `http://localhost:8080/metrics` 可以查看其暴露给 Prometheus 的所有数据：

![/images/cadvisor/cadvisor-05.png](/images/cadvisor/cadvisor-05.png)

### 总结：

cadvisor 是一个非常好的容器数据收集的工具，它提供良好的 Web 界面，方便查询。

既然 cadvisor 已经将所有容器的数据收集了，那接下来就是结合 Prometheus 和 Grafana 做监控和告警了，这部分的内容我们将在后面的文章中详细讲解，敬请期待。
