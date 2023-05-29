---
title: "containerd 启动报错解决"
subtitle: "undefined symbol: seccomp_version 问题诊断"
date: 2020-06-04 17:03:04+0800
# cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [containerd]
# gh-repo: daattali/beautiful-jekyll
# gh-badge: [star, fork, follow]
# comments: true
---

最近开始研究 containerd, 按照官网教程 在 Centos7.2 上下载了 [containerd-1.3.4.linux-amd64.tar.gz](https://github.com/containerd/containerd/releases/download/v1.3.4/containerd-1.3.4.linux-amd64.tar.gz) ，解压缩运行程序的时候遇到如下报错信息：

```
./containerd: symbol lookup error: ./containerd: undefined symbol: seccomp_version
```

网上查了下，这个错误意思是说系统缺少对应版本的 `libseccomp`， 有两个可能：

- 系统没有安装 libseccomp
- 系统安装的 libseccomp 版本不匹配（较低），建议使用 2.4.2

### libseccomp 是什么

> library provides an easy to use, platform independent, interface to the Linux Kernel's syscall filtering mechanism.

它是一个 C 语言开发的 Linux 内核系统调用过滤帮助库，很多容器项目都使用到它，比如 containerd、docker、runc ，其源代码地址为 [https://github.com/seccomp/libseccomp](https://github.com/seccomp/libseccomp)。

### libseccomp 安装

因为 containerd 需要依赖的 libseccomp 较新，经过测试，v2.4.2 可以在 Centos7.2 成功运行，所以我们直接通过源代码来安装。

- 下载源代码

```
curl -LO https://github.com/seccomp/libseccomp/releases/download/v2.4.2/libseccomp-2.4.2.tar.gz
```

- 解压缩

```
tar -vxf libseccomp-2.4.2.tar.gz
cd libseccomp-2.4.2
```

- 编译构建

```
./configure --prefix=/usr --disable-static &&  make 
```

- 安装

```
make install 
```

- 通过 `whereis libseccomp` 查看安装目录

```
libseccomp: /usr/lib/libseccomp.so /usr/lib/libseccomp.la
```

> 注意：如果以前使用系统包管理工具安装了低版本的 libseccomp，即使通过编译安装最新版本，还是会报错。

这是因为引用路径的优先级的缘故，其 containerd 还是使用了老版本，故需要先删除系统包管理安装的老版本，删除命令根据操作系统的类别而定，比如 Centos 可以使用 `yum remove libseccomp`。


### 再次运行 containerd

当安装了 v2.4.2 的 libseccomp 后， 我们再次运行 containerd，报错信息消失。

```
INFO[2020-06-04T17:47:34.306847085+08:00] starting containerd                           revision=814b7956fafc7a0980ea07e950f983d0837e5578 version=v1.3.4
INFO[2020-06-04T17:47:34.361473441+08:00] loading plugin "io.containerd.content.v1.content"...  type=io.containerd.content.v1
INFO[2020-06-04T17:47:34.361589154+08:00] loading plugin "io.containerd.snapshotter.v1.btrfs"...  type=io.containerd.snapshotter.v1
INFO[2020-06-04T17:47:34.362042869+08:00] skip loading plugin "io.containerd.snapshotter.v1.btrfs"..
```

参考资料：

- [containerd/issues/4008](https://github.com/containerd/containerd/issues/4008)
- [libseccomp 代码库](https://github.com/seccomp/libseccomp)
- [libseccomp 安装](http://www.linuxfromscratch.org/blfs/view/svn/general/libseccomp.html)
- [libseccomp 介绍](http://blog.fpliu.com/it/software/libseccomp)