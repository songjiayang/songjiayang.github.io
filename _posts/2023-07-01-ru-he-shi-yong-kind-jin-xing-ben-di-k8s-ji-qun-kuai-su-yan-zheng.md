---
title: "如何使用 kind 进行本地 k8s 集群快速验证"
subtitle: 
date: 2023-07-01 22:18:51+0800
thumbnail-img: /images/kind/01/diagram.png
# cover-img: 
# share-img: 
tags: [kubernetes, kind]
---

以前我们在进行 K8s 本地环境快速验证的时候，第一选择往往是 minikube，但其实社区还有另外两个方案（Kind 和 k3d），今天我们就来来讲解一下 Kind 的用法。

本文内容包括:

- Kind 基本介绍
- Kind 安装
- 集群管理
- kubectl 配置
- 简单示例

## Kind 基本介绍

![diagram.png](/images/kind/01/diagram.png)

Kind 是 Kubernetes IN Docker 的字母缩写，它依赖 Docker（当然也支持 Podman），相对 Minikube 而言，它依赖更轻量，资源占用更少，集群创建销毁速度更快。

故它的卖点就是简单、非常适合本地 K8s 集群自身和 K8s 项目的集成测试。

环境依赖：

- Docker: 20.10 及以上, 安装请参考 [docker install docs](https://docs.docker.com/engine/install/)
- Podman: 3.0 及以上, 安装请参考 [podman install docs](https://podman.io/docs/installation)

备注： Docker 和 Podman 安装一个即可。

## Kind 安装

Kind 的安装方式非常简单，大家可以参考[安装文档](https://kind.sigs.k8s.io/docs/user/quick-start/#installation),这里我直接使用编译好的二进制文件进行安装。

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

然后执行 `kind --help` 确定是否安装成功。

## 集群管理

- 默认创建 

```
kind create cluster

Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

默认创建的集群名为 kind，它只包含一个控制面节点。

- 自定义创建

我们可以通过配置文件定义我们集群的节点数，比如 1个控制平面，3个 worker 节点，my-cluster.yaml 内容如下：

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
```

然后再通过 --config 指定使用它，使用 --name 也可以修改集群名词。

```
kind create cluster --name my-cluster --config ./my-cluster.yaml
```

- 查看集群列表 

```
kind get clusters
```

- 删除集群 

```
kind delete cluster -n my-cluster
kind delete clusters my-cluster
```

## kubectl 配置

当 cluster 集群创建成功后，可以直接使用 kubectl 对集群进行操作，第一步就是要对 kubectl 进行配置。

- 查看所有集群 

```
kubectl config get-contexts
```

- 设置 kubectl 默认集群

```
kubectl config set-context kind-my-cluster
```

- 查看集群信息

```
kubectl cluster-info
```

## 简单示例

我们以一个最简单的 nginx 部署为例，看看 Kind 部署的 K8s 集群工作是否正常。


nginx.yaml 配置内容为：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

- 部署 nginx

```
kubectl apply -f nginx.yaml
```

- 查看部署状态

```
kubectl get deployment nginx-deployment


NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           118s
```

- 查看对应 pods

```
kubectl get pods --selector='app=nginx'


NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-cbdccf466-nzfl7   1/1     Running   0          4m36s
nginx-deployment-cbdccf466-qvgtg   1/1     Running   0          4m36s
```

可以看到使用 kubectl 能够正常部署应用程序到使用 Kind 部署的本地 K8s 集群中。

## 总结

在使用 Kind 进行本地 K8s 验证时，其安装和使用上明显比 Minikube 简单，它运行速度也更快，占用资源也更少，更适合日常本地系统集成测试使用。

接下来笔者准备深入了解 Kind 的实现原理，看它是如何进行集群创建、镜像管理、Pods 创建和删除的，学习心得也会不定期更新到博客，敬请期待。