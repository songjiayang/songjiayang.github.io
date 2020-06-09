---
title: "如何在 Go 中优雅关闭子进程"
date: 2019-05-31 14:51:06+0800
tags: [go]
---

![golang-fork.png](/images/golang-fork.png)

## 如何在 Go 中运行一个外部命令

有时我们会遇到这样的需求，在一个主进程中启动另外一个进程，而在 Go 中可以使用 exec 包的 `Cmd` 来轻松实现这类需求，例如代码：

```golang
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
	"os/signal"
)

func main() {
	cmd := exec.Cmd{
		Path: "nc",
		Args: []string{"-u", "-l", "8888"},
		Dir:  "/usr/bin",
	}

	if err := cmd.Start(); err != nil {
		log.Panic(err)
	}

	fmt.Println("Start child process with pid", cmd.Process.Pid)

	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)
	s := <-c
	fmt.Println("Got signal:", s)
}
```
 
这段代码的含义是： 使用 `nc -u -l 8888` 来模拟一个常驻进程，然后通过 Go 的 `exec.Cmd` 来运行它，并且 Go 代码不退出，运行代码：

```bash
$ go run main.go

Start child process with pid 35904
```

输出结果表明我们已经通过 Go 成功调用外部命令，起了一个子进程，其进程号为 `35904`，我们还可以通过命令 `ps -ef 35904` 来确认：

```bash
 UID   PID  PPID   C STIME   TTY           TIME CMD
2062309935 35904 35903   0  3:36PM ttys008    0:00.00 -u -l 8888
``` 

### 如何结束子进程

- 首先想到的就是 `kill` 命令，尝试使用 `kill 35904`

```bash
$ kill 35904
$ ps -ef 35904

UID   PID  PPID   C STIME   TTY           TIME CMD
2062309935 35904 35903   0  3:36PM ttys008    0:00.00 (nc)
```

发现 kill 命令并不好用，进程还在，然后换成  `kill -9` 也同样不起作用。不过该进程已经停止运行了，可以看到监听由 `0:00.00 -u -l 8888` 变成了 `0:00.00 (nc)`, 不再监听 8888 端口，只是进程资源还没释放而已。 

- 使用 Go 代码结束该进程

因为 Go 的 Cmd 内置了 `Process.Kill()` 函数，我们可以尝试使用它来关闭子进程，修改代码，添加如下内容：

```
// After five second, kill cmd's process
time.Sleep(5 * time.Second)
cmd.Process.Kill()
```

重新运行代码，发现 5 秒过后，该子进程还在。其实调用 `cmd.Process.Kill()` 和外部使用 kill 命令是一样的，父进程还没有释放资源，所以子进程不能清理完成。

- 使用 `cmd.Wait()` 完成资源清理，修改后的完整代码如下：

```golang
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
	"os/signal"
	"time"
)

func main() {
	cmd := exec.Cmd{
		Path: "nc",
		Args: []string{"-u", "-l", "8888"},
		Dir:  "/usr/bin",
	}

	if err := cmd.Start(); err != nil {
		log.Panic(err)
	}

	fmt.Println("Start child process with pid", cmd.Process.Pid)

	// Wait releases any resources associated with the Cmd
	go func() {
		if err := cmd.Wait(); err != nil {
			fmt.Printf("Child process %d exit with err: %v\n", cmd.Process.Pid, err)
		}
	}()

	// After five second, kill cmd's process
	time.Sleep(5 * time.Second)
	cmd.Process.Kill()

	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)
	s := <-c
	fmt.Println("Got signal:", s)
}
```

运行代码，可以得到如下结果：

```golang
$ go run main.go

Start child process with pid 41666
Child process 41666 exit with err: signal: killed
```


再通过 `ps -el 41666` 命令确认子进程 `41666` 已不存在。

### 结语

Go 中 exec.Cmd 封装的很好，对于外部命令调用非常方便，但是使用它的时候，需要注意对子进程的资源进行释放，其关键函数就是 `cmd.Wait()`, 
所以用到 cmd 的地方，一定添加 `cmd.Wait()` 的逻辑。

----

参考链接：

* [https://golang.org/pkg/os/exec/#Cmd.Wait](https://golang.org/pkg/os/exec/#Cmd.Wait)


