---
title: "Unit Test In Go With Docker"
date: 2019-04-02 18:10:29+0800
tags: [docker, go, 云原生]
---

![docker-testing.png](/images/docker-testing.png)

本篇文章主要讲解如何在 Docker 中进行 Go 单元测试，依赖 Docker 和 Go Modules。

### 为什么是 Docker

在 Docker 之前我们往往需要在 Jenkins 服务器上配置不同的 Go 版本以及针对每个项目配置 GOPATH，项目之间的隔离性差，经常出现基础库版本冲突的问题。

有了 Docker，我们可以在不同容器中运行单元测试，该测试不局限不同项目，甚至可以是同一项目不同分支。

所以在测试隔离性和项目测试并发度上都有很大提升，而且测试结束后，环境清理也简单许多。

### 为什么是 Go Modules

Go Modules 作为官方默认的包管理工具，基本解决了 Go 长期存在的包管理问题，它为我们的项目管理带来很多好处：

- 自动解析和添加依赖
- 签名验证
- 依赖缓存
- 支持相对路径依赖
- 支持依赖一键打包，方便在离线环境下运行程序

### 实际例子

下面我们来看一个简单例子，来自 [Gin Testing Example](https://gin-gonic.com/docs/testing/)，项目目录结构为：

```
$ tree .
.
├── go.mod
├── go.sum
├── main.go
└── main_test.go

0 directories, 4 files
```

main.go 内容：

```
package main

func setupRouter() *gin.Engine {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})
	return r
}

func main() {
	r := setupRouter()
	r.Run(":8080")
}
```

main_test.go 内容：

```
package main

import (
	"net/http"
	"net/http/httptest"
	"testing"

	"github.com/stretchr/testify/assert"
)

func TestPingRoute(t *testing.T) {
	router := setupRouter()

	w := httptest.NewRecorder()
	req, _ := http.NewRequest("GET", "/ping", nil)
	router.ServeHTTP(w, req)

	assert.Equal(t, 200, w.Code)
	assert.Equal(t, "pong", w.Body.String())
}
```


首先尝试在主机执行 `go test ./...` ，测试通过，下面我们想办法在 Docker 中进行测试。


### Docker 运行测试

步骤1: 对项目依赖进行打包，方便在 Docker 无网环境下运行单元测试

```
go mod vendor
```

此时所有的依赖都打包放在项目根目录的 vendor 下面：

```bash
$ tree .
.
├── go.mod
├── go.sum
├── main.go
├── main_test.go
└── vendor
    ├── github.com
    │   ├── davecgh
    │   │   └── go-spew
    │   │       ├── LICENSE
    │   │       └── spew
    │   │           ├── bypass.go
    │   │           ├── bypasssafe.go
    │   │           ├── common.go
```


步骤2: 制作 Go Docker 镜像

因为我们只需要 Go 的标准环境，所以可以使用官方的标准镜像，例如 golang:1.12.1。

如果你有特殊需求，可以定制自己的 Go 镜像, 例如自定义启动命令，添加数据库依赖等：

- Dockerfile:

```
FROM ubuntu
RUN apt-get update && apt-get install -y libssl1.0.0 libssl-dev gcc

RUN mkdir -p /data/db /opt/go/ /opt/gopath

# go1.xx.x.linux-amd64.tar.gz 解压缩后为 go
ADD go /opt/go 
RUN cp /opt/go/bin/* /usr/local/bin/
ENV GOROOT=/opt/go GOPATH=/opt/gopath

WORKDIR /ws
CMD GOPROXY=off go test -mod=vendor ./...
```

- 执行 docker build:

```
docker build . -t gotesting:v0.0.1
```


步骤3: 在 Docker 中运行 Golang 测试

- 使用 golang:1.12.1:

```
$ docker run --volume=$(pwd):/ws \
     --workdir=/ws golang:1.12.1 \
    /bin/bash -c "GOPROXY=off go test -mod=vendor ./..."
ok  	github.com/songjiayang/gin-test	0.011s
```

- 使用自定义镜像 gotesting:v0.0.1:

```
docker run --volume=$(pwd):/ws gotesting:v0.0.1
ok  	github.com/songjiayang/gin-test	0.014s
```

可以看到两种方式都能在 Docker 中运行 Go 单元测试，自定义镜像可以指定默认运行命令，使操作更简单，上面命令的含义是：

- --volume=$(pwd):/ws ：将项目根目录映射到 Docker 容器中的 /ws 目录。
- --workdir=/ws ： 指定容器运行工作目录为 /ws, 即映射外面项目根目录。
- /bin/bash -c "GOPROXY=off go test -mod=vendor ./..." : 表示在 Docker 中以离线，vendor 模式运行 Go 单元测试。

### 做的更好

到目前为止，我们已经掌握在 Docker 中运行 Go 单元测试的方法。当然我们还可以做的更好，Docker 运行的标准输出可以和 Jenkins 天然集成，实现项目的自动测试。