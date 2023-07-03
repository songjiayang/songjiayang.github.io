---
title: "为 Kind 本地 K8s 集群添加 Dashboard"
subtitle: 
date: 2023-07-02 20:58:29+0800
thumbnail-img: /images/kind/02/dashboard-token.png
# cover-img: 
# share-img: 
tags: [kubernetes, kind]
---

在上一篇文章中，我们讲解了如何通过 Kind 快速部署本地 K8s 环境，今天我们将进一步讲解其本地 K8s 环境中添加 K8s Dashboard 的方法。

Kubenetes Dashboard 项目地址为 [https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)， 可以看到它提供统一的 Web 界面，对 K8s 集群中的不同命令空间的各种对象资源进行观测，非常简约直观，下面我们就来讲解其具体用法。

## Kubenetes dashboard 安装

- Helm 安装

```
helm repo add kubernetes-dashboard --insecure-skip-tls-verify https://kubernetes.github.io/dashboard/
helm install dashboard kubernetes-dashboard/kubernetes-dashboard -n kubernetes-dashboard --create-namespace --insecure-skip-tls-verify
```

此时当您看到类似信息，说明安装成功：

```
kip-tls-verify
NAME: dashboard
LAST DEPLOYED: Mon Jul  3 17:15:03 2023
NAMESPACE: kubernetes-dashboard
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
*********************************************************************************
*** PLEASE BE PATIENT: kubernetes-dashboard may take a few minutes to install ***
*********************************************************************************

Get the Kubernetes Dashboard URL by running:
.....
```

-  Manifest 方法 安装

除了 helm 部署方法外，我们还可以使用 Manifest 进行安装，命令为：

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

说明：目前官方更推荐大家使用 Helm 安装。

## 访问 Dashboard 

- 通过 port-forward 访问

当我们使用 Helm 安装完成后，通过提示信息可以看到相关方法，即

```
export POD_NAME=$(kubectl get pods -n kubernetes-dashboard -l "app.kubernetes.io/name=kubernetes-dashboard,app.kubernetes.io/instance=dashboard" -o jsonpath="{.items[0].metadata.name}")
echo https://127.0.0.1:8443/
kubectl -n kubernetes-dashboard port-forward $POD_NAME 8443:8443
```

执行该命令后访问 [https://127.0.0.1:8443/](https://127.0.0.1:8443/) 链接。

- 通过 kubectl proxy 

先执行 `kubectl proxy` 命令，再访问页面 [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:dashboard-kubernetes-dashboard:https/proxy/#/login](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:dashboard-kubernetes-dashboard:https/proxy/#/login) 。


注意：在本地进行测试验证，这步可能等待时间较长。

无论通过哪种方案，默认都会进入权限验证页面，类似：

![dashboard-token.png](/images/kind/02/dashboard-token.png)

## 创建 token

参考文档 [access-control](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/README.md)。

- 添加 admin-user 账户并设置访问权限

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  
---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

- 生成 token

```
kubectl -n kubernetes-dashboard create token admin-user
```

将生成的 token 拷贝到 Dashboard 认证页面，点击确认即可自动跳转默认首页。

![dashboard-index.png](/images/kind/02/dashboard-index.png)

## 总结

在 Kind 新建的 k8s 环境中，我们可以通过 Helm 或者 Manifest 方式安装 Kubenetes Dashboard，然后配置对应的授权账户即可进行页面访问，整个过程还是比较简单的。