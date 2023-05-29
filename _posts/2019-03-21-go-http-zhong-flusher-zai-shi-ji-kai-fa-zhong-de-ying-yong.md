---
title: "Go http.Flusher 在实际项目中的应用"
date: 2019-03-21 20:16:31+0800
tags: [go]
---

![cover.png](/images/flush/cover01.png)


最近在使用 Docker Go SDK 做开发的时候，参考了官方的[示例](https://docs.docker.com/develop/sdk/examples/)代码:

```golang
package main

import (
	"io"
	"os"

	"github.com/docker/docker/api/types"
	"github.com/docker/docker/api/types/container"
	"github.com/docker/docker/client"
	"golang.org/x/net/context"
)

func main() {
	ctx := context.Background()
	cli, err := client.NewEnvClient()
	if err != nil {
		panic(err)
	}

	reader, err := cli.ImagePull(ctx, "docker.io/library/alpine", types.ImagePullOptions{})
	if err != nil {
		panic(err)
	}
	io.Copy(os.Stdout, reader)

	resp, err := cli.ContainerCreate(ctx, &container.Config{
		Image: "alpine",
		Cmd:   []string{"echo", "hello world"},
		Tty:   true,
	}, nil, nil, "")
	if err != nil {
		panic(err)
	}

	if err := cli.ContainerStart(ctx, resp.ID, types.ContainerStartOptions{}); err != nil {
		panic(err)
	}

	statusCh, errCh := cli.ContainerWait(ctx, resp.ID, container.WaitConditionNotRunning)
	select {
	case err := <-errCh:
		if err != nil {
			panic(err)
		}
	case <-statusCh:
	}

	out, err := cli.ContainerLogs(ctx, resp.ID, types.ContainerLogsOptions{ShowStdout: true})
	if err != nil {
		panic(err)
	}

	io.Copy(os.Stdout, out)
}
```  

这段代码主要做了 4 件事：

1. 拉取镜像
2. 创建容器
3. 启动容器
4. 获取容器日志输出


其中有个地方引起了我的注意：

```
reader, err := cli.ImagePull(ctx, "docker.io/library/alpine", types.ImagePullOptions{})
if err != nil {
    panic(err)
}
io.Copy(os.Stdout, reader)
```
 
因为没有用到返回结果 `reader`，所以不假思素地将它去掉：

```
_, err := cli.ImagePull(ctx, "docker.io/library/alpine", types.ImagePullOptions{})
if err != nil {
    panic(err)
}
```  

保存代码，再次执行，当运行到 `cli.ContainerCreate` 的时候，会收到`No such image: xx`之类的错误。

能够运行到  `cli.ContainerCreate` 说明已经执行了 `cli.ImagePull`，而且没有错误，那为何还会提示找不到镜像？

想到的唯一解释是： 

> 当通过 `cli.ImagePull` 拉取镜像的时候，虽然请求返回了结果，并不表示 Docker 服务端真的将镜像拉取完成，因为从远程仓库拉取镜像往往耗时较长，很有可能正在拉取中。

为了验证猜想，我们深入 Docker 源码，一探究竟。

## Docker 源码探究

### 客户端代码：

当客户端执行 `ImagePull` 的时候，实际发送了一个 `POST` 请求，地址为 `/images/create`,  可以参考[代码](https://github.com/moby/moby/blob/master/client/image_create.go#L34) :

![flush05.png](/images/flush/flush05.png)

### 服务端代码：

/images/create 请求处理[代码](https://github.com/moby/moby/blob/master/api/server/router/image/image_routes.go#L26)为:

![ImagePull.png](/images/flush/flush01.png)


> 这段代码主要通过 `output   = ioutils.NewWriteFlusher(w)` 封装一个 `WriteFlusher` 返回结果。

WriteFlusher 的 [Write 函数](https://github.com/moby/moby/blob/8e610b2b55bfd1bfa9436ab110d311f5e8a74dcb/pkg/ioutils/writeflusher.go#L26) 为 :

![flush02.png](/images/flush/flush02.png)


最后请求是通过 `progressOutput` 的 [WriteProgress](https://github.com/moby/moby/blob/a3f54d45709b3b48022fafd3c4796b1088be1b9d/pkg/streamformatter/streamformatter.go#L115) 返回进度：

![flush03.png](/images/flush/flush03.png)

到目前为止我们大致弄明白了 Docker `ImagePull` 的逻辑，简单总结一下：

1. 客户端发送 `POST` 请求到 Docker 服务端  `/images/create`。
2. 服务端通过  `WriteFlusher` 来负责请求的返回，这里使用了 Go http.Flusher， 可以不断向客户端刷新数据。

所以当我们客户端发送 `ImagePull` 请求后，虽然可以很快获得 http.Response 对象，但它并不表示最终任务完成（请求结束），而是先返回请求状态码，再不断返回拉取进度，那怎样才知道任务完成了呢？

可以使用官方例子中的 `io.Copy(os.Stdout, reader)` 或 `ioutil.ReadAll(reader)` ，因为它们读取 Body 内容，会阻塞在这里，直到任务结束（读取报错，或者 EOF 标记）。


我把 Docker API 简单梳理了下，大致可以分为两类：

- 使用 WriteFlusher 返回结果：主要用于耗时的任务，不仅可以通过快速向客户端返回状态码（200）来表明请求合法已经被接受处理，还可以不断向客户端刷新任务进度，实现实时效果，类似 Pusher。
- 直接返回结果：主要用于耗时较短的状态查询接口。

## Go 中 http.Flusher 示例

在某些场景下 Go 的 `http.Flusher`  还是非常有用的，比如可以用它来做流式 IO，如文件上传/下载/内容预处理等，那我们来看一个简单 http.Flusher 的用法：

```golang 
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("x-request-id", "x-request-id")

		f, _ := w.(http.Flusher)

		for i := 0; i < 10; i++ {
			fmt.Fprintf(w, "time.now(): %v \n\r", time.Now())
			f.Flush()
			time.Sleep(time.Second)
		}

	})

	log.Fatal(http.ListenAndServe(":8888", nil))
}

```

> 代码逻辑为： 每隔一秒向客户端  flush 当前时间。

当运行代码，并使用 `curl -i http://localhost:8888` 查看结果，可以看到类似输出：

![flush04.png](/images/flush/flush04.png)

> 注意：想要看到整个刷新过程，需要客户端的支持 (这里使用的是 curl, 你还可以使用 wget），我们可以看到终端每隔一秒从服务端获取输出结果，10 秒后请求结束。

在调用 Flush 之前，需要保证写入 `http.ResponseWriter` 的内容以 `\n` 结尾，不然不会输出到客户端。


## 写在最后

我们通过 Docker 客户端 `ImagePull` 接口一个问题出发，通过 Docker 源码了解了整个 Docker 镜像拉取流程，
也弄明白使用 `io.Copy(os.Stdout, reader)` 确保拉取任务结束的必要性。

举一反三，根据这个实现逻辑，我们可以将 Docker API 分为使用 WriteFlusher 和直接返回 body 内容两类。

最后还通过简单例子学习了 Go 的 `http.Flusher` 用法，希望在实际的工作中对大家有所帮助。


