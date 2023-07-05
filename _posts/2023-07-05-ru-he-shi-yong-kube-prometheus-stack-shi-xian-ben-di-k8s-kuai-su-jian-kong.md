---
title: "如何使用 kube-prometheus-stack 实现本地 K8s 快速监控"
subtitle: 
date: 2023-07-05 21:11:24+0800
# thumbnail-img: 
# cover-img: 
# share-img: 
tags: [kubernetes, kind]
---

上一篇文章中，我们讲到如何安装 kube dashboard 对我们 k8s 环境的对象运行状况进行观测，今天我们将讲解如何使用 kube-prometheus-stack 对其进行监控。

## kube-prometheus-stack 安装

这里我们直接使用 helm 安装，文档地址参考 [https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)。

主要命令如下：

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm show values prometheus-community/kube-prometheus-stack > ./values.yaml # 更改默认配置，比如抓取、告警、以及持久卷等。
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack -f values.yaml -n monitoring
```

备注： 如果遇到 x509的问题，可以使用参数 --insecure-skip-tls-verify 忽略证书校验。

## k8s 服务运行状态

当命令运行成功后，我们已经在 monitoring 命令空间下部署了 kube-prometheus-stack，当我们执行 `kubectl get all -n monitoring`，可以看到所有的相关对象运行情况如下：

```
# pods
NAME                                                            READY   STATUS    RESTARTS        AGE
pod/alertmanager-kube-prometheus-stack-alertmanager-0           2/2     Running   2 (4h21m ago)   19h
pod/kube-prometheus-stack-grafana-65df74f4fb-7t4k7              3/3     Running   3 (4h21m ago)   19h
pod/kube-prometheus-stack-kube-state-metrics-57dc5579bd-4v97f   1/1     Running   2 (4h20m ago)   19h
pod/kube-prometheus-stack-operator-75b7d5b98d-nxkft             1/1     Running   2 (4h19m ago)   19h
pod/kube-prometheus-stack-prometheus-node-exporter-2545c        1/1     Running   1 (4h21m ago)   19h
pod/kube-prometheus-stack-prometheus-node-exporter-4twsb        1/1     Running   1 (4h21m ago)   19h
pod/kube-prometheus-stack-prometheus-node-exporter-z6bff        1/1     Running   1 (4h21m ago)   19h
pod/prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   2 (4h21m ago)   19h

# services
NAME                                                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-operated                            ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   19h
service/kube-prometheus-stack-alertmanager               ClusterIP   10.96.223.171   <none>        9093/TCP,8080/TCP            19h
service/kube-prometheus-stack-grafana                    ClusterIP   10.96.209.142   <none>        80/TCP                       19h
service/kube-prometheus-stack-kube-state-metrics         ClusterIP   10.96.173.162   <none>        8080/TCP                     19h
service/kube-prometheus-stack-operator                   ClusterIP   10.96.156.60    <none>        443/TCP                      19h
service/kube-prometheus-stack-prometheus                 ClusterIP   10.96.33.206    <none>        9090/TCP,8080/TCP            19h
service/kube-prometheus-stack-prometheus-node-exporter   ClusterIP   10.96.49.229    <none>        9100/TCP                     19h
service/prometheus-operated                              ClusterIP   None            <none>        9090/TCP                     19h

# daemonset
NAME                                                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-prometheus-stack-prometheus-node-exporter   3         3         3       3            3           kubernetes.io/os=linux   19h

# deployment
NAME                                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kube-prometheus-stack-grafana              1/1     1            1           19h
deployment.apps/kube-prometheus-stack-kube-state-metrics   1/1     1            1           19h
deployment.apps/kube-prometheus-stack-operator             1/1     1            1           19h

# replicaset
NAME                                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/kube-prometheus-stack-grafana-65df74f4fb              1         1         1       19h
replicaset.apps/kube-prometheus-stack-kube-state-metrics-57dc5579bd   1         1         1       19h
replicaset.apps/kube-prometheus-stack-operator-75b7d5b98d             1         1         1       19h

# statefulset
NAME                                                               READY   AGE
statefulset.apps/alertmanager-kube-prometheus-stack-alertmanager   1/1     19h
statefulset.apps/prometheus-kube-prometheus-stack-prometheus       1/1     19h
```

通过最终部署的 pods 可以看到，默认 kube-prometheus-stack 配置一共包含：
- 1 个 Prometheus server
- 3 个 node exporter， 因为本地 kind 部署 k8s 集群有1个控制面节点，2个 Worker 节点
- 1 个 alertmanager
- 1 个 Grafana
- 1 个 kube-state-metrics 和 一个 kube-prometheus-stack-operator

## 查看 Grafana 和 Prometheus 控制台

因为默认配置没有启用 ingress，所以我们可以通过修改 service 的 type 为 NodePort 方式，然后使用 node 的IP+端口进行访问。

- 修改 grafana service 的配置

```
kubectl edit svc/kube-prometheus-stack-grafana -n monitoring

==> 
spec:
  ports:
    port: 80
    protocol: TCP
    targetPort: 3000
    nodePort: 30030 # 添加对应 Node节点访问端口
   ....
  type: NodePort  # 修改为 NodePort 类型

```

此时通过<控制面节点IP>:30030 即可进入 grafana 页面，默认密码为 admin/prom-operator。


- 修改 prometheus service 的配置

```
kubectl edit svc/kube-prometheus-stack-prometheus -n monitoring
```

同理修改为 NodePort 类型，并指定响应端口即可。

## 简单总结

到目前为止，我们就通过 kube-prometheus-stack 快速搭建了 Prometheus 监控环境，除了使用 heml 之外，我还可以使用 Prometheus Operator 的 [manifests](https://prometheus-operator.dev/docs/prologue/quick-start/) 进行安装，它默认是多实例部署。