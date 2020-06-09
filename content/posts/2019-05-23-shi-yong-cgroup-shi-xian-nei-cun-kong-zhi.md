---
title: "如何在 Go 中使用 CGroup 实现进程内存控制"
date: 2019-05-23 18:12:57+0800
tags: [go]
---

![lxc.png](/images/cgroup/lxc.png)

## 背景

从 Linux 内核 2.6.25 开始，`CGroup` 支持对进程内存的隔离和限制，这也是 Docker 等容器技术的底层支撑。

使用 CGroup 有如下好处：

- 在共享的机器上，进程相互隔离，互不影响，对其它进程是种保护。
- 对于存在内存泄漏的进程，可以设置内存限制，通过系统 OOM 触发的 Kill 信号量来实现重启。

## CGroup 快速入门

### 默认挂载分组

Linux 系统默认支持 `CGroup`, 而且默认挂载所有选项，可以使用 `mount -t cgroup` 来查看：

```bash
$ mount -t cgroup

cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/net_cls type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
```

`CGroup` 相关的资源包括 `cpu`,`memory`,`blkio`等，而我们今天主要关心的是内存，即 `/sys/fs/cgroup/memory`。

### 创建 `climits` 内存分组

我们可以使用 `mkdir /sys/fs/cgroup/memory/climits` 来创建属于自己的内存组 `climits`:

```
$ mkdir /sys/fs/cgroup/memory/climits
```

此时系统已经在目录 `/sys/fs/cgroup/memory/climits` 下为我们生成了内存相关的所有配置：

```
$ ls -la /sys/fs/cgroup/memory/climits

cgroup.clone_children  memory.kmem.limit_in_bytes          memory.kmem.tcp.usage_in_bytes  memory.memsw.max_usage_in_bytes  memory.soft_limit_in_bytes  tasks
cgroup.event_control   memory.kmem.max_usage_in_bytes      memory.kmem.usage_in_bytes      memory.memsw.usage_in_bytes      memory.stat
cgroup.procs           memory.kmem.slabinfo                memory.limit_in_bytes           memory.move_charge_at_immigrate  memory.swappiness
memory.failcnt         memory.kmem.tcp.failcnt             memory.max_usage_in_bytes       memory.numa_stat                 memory.usage_in_bytes
memory.force_empty     memory.kmem.tcp.limit_in_bytes      memory.memsw.failcnt            memory.oom_control               memory.use_hierarchy
memory.kmem.failcnt    memory.kmem.tcp.max_usage_in_bytes  memory.memsw.limit_in_bytes     memory.pressure_level            notify_on_release
```

主要配置含义：

- cgroup.procs: 使用该组配置的进程列表。
- memory.limit_in_bytes：内存使用限制。
- memory.memsw.limit_in_bytes：内存和交换分区总计限制。 
- memory.swappiness: 交换分区使用比例。
- memory.usage_in_bytes： 当前进程内存使用量。
- memory.stat: 内存使用统计信息。
- memory.oom_control: OOM 控制参数。
- 其它，参考[官方手册](http://man7.org/linux/man-pages/man7/cgroups.7.html)

### 设置内存限制

假设有进程 pid `1234`，希望设置内存限制为 10MB，我们可以这样操作：

- limit_in_bytes 设置为 10MB 

```
echo 10M > /sys/fs/cgroup/memory/climits/memory.limit_in_bytes
```

- swappiness 设置为 0，表示禁用交换分区，实际生产中可以配置合适的比例。

```
echo 0 > /sys/fs/cgroup/memory/climits/memory.swappiness
```

- 添加控制进程

```
echo 1234 > /sys/fs/cgroup/memory/climits/cgroup.procs
```

当进程 `1234` 使用内存超过 10MB 的时候，默认进程 `1234` 会触发 OOM，被系统 Kill 掉。

## Go 实现进程内存限制

上面我们已经讲到 CGroup 内存限制的原理，接下来我们就用 Go 代码来实现一个简单的进程内存限制以及守护（被 Kill 能够自动重启）。

- 进程测试代码:

该代码主要逻辑是每隔一秒申请 1MB 存储空间，并且不释放，然后再打印下 Go 的内存申请情况。

```go
// example/simple_app.go
package main

import (
	"fmt"
	"os"
	"runtime"
	"time"
)

const (
	MB = 1024 * 1024
)

func main() {
	blocks := make([][MB]byte, 0)
	fmt.Println("Child pid is", os.Getpid())

	for i := 0; ; i++ {
		blocks = append(blocks, [MB]byte{})
		printMemUsage()
		time.Sleep(time.Second)
	}
}

func printMemUsage() {
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
	fmt.Printf("Alloc = %v MiB", bToMb(m.Alloc))
	fmt.Printf("\tSys = %v MiB \n", bToMb(m.Sys))
}

func bToMb(b uint64) uint64 {
	return b / MB
}
```

通过 GOOS=linux GOARCH=amd64 go build -o simpleapp example/simple_app.go 命令，编译一个 Linux 版本的可执行程序 `simpleapp`。

### 进程守护程序

该守护程序主要实现进程内存限制和进程守护（自动重启），代码如下：

```go
// main.go
package main

import (
	"flag"
	"fmt"
	"io/ioutil"
	"log"
	"os"
	"os/exec"
	"os/signal"
	"path/filepath"
	"syscall"
)

var (
	rssLimit   int
	cgroupRoot string
)

const (
	procsFile       = "cgroup.procs"
	memoryLimitFile = "memory.limit_in_bytes"
	swapLimitFile   = "memory.swappiness"
)

func init() {
	flag.IntVar(&rssLimit, "memory", 10, "memory limit with MB.")
	flag.StringVar(&cgroupRoot, "root", "/sys/fs/cgroup/memory/climits", "cgroup root path")
}

func main() {
	flag.Parse()

	// set memory limit
	mPath := filepath.Join(cgroupRoot, memoryLimitFile)
	whiteFile(mPath, rssLimit*1024*1024)

	// set swap memory limit to zero
	sPath := filepath.Join(cgroupRoot, swapLimitFile)
	whiteFile(sPath, 0)

	go startCmd("./simpleapp")

	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)
	s := <-c
	fmt.Println("Got signal:", s)
}

func whiteFile(path string, value int) {
	if err := ioutil.WriteFile(path, []byte(fmt.Sprintf("%d", value)), 0755); err != nil {
		log.Panic(err)
	}
}

type ExitStatus struct {
	Signal os.Signal
	Code   int
}

func startCmd(command string) {
	restart := make(chan ExitStatus, 1)

	runner := func() {
		cmd := exec.Cmd{
			Path: command,
		}

		cmd.Stdout = os.Stdout

		// start app
		if err := cmd.Start(); err != nil {
			log.Panic(err)
		}

		fmt.Println("add pid", cmd.Process.Pid, "to file cgroup.procs")

		// set cgroup procs id
		pPath := filepath.Join(cgroupRoot, procsFile)
		whiteFile(pPath, cmd.Process.Pid)

		if err := cmd.Wait(); err != nil {
			fmt.Println("cmd return with error:", err)
		}

		status := cmd.ProcessState.Sys().(syscall.WaitStatus)

		options := ExitStatus{
			Code: status.ExitStatus(),
		}

		if status.Signaled() {
			options.Signal = status.Signal()
		}

		cmd.Process.Kill()

		restart <- options
	}

	go runner()

	for {
		status := <-restart

		switch status.Signal {
		case os.Kill:
			fmt.Println("app is killed by system")
		default:
			fmt.Println("app exit with code:", status.Code)
			return
		}

		fmt.Println("restart app..")
		go runner()
	}
}
```

这段代码的主要逻辑为：

- 通过配置参数 memory ，修改 memory.limit_in_bytes 和 memory.swappiness 来设置最大内存使用量。
- 通过 cmd.Start() 启动一个进程。
- 将新生成的进程号 cmd.Process.Pid 写到 cgroup.procs。
- 通过 cmd.Wait() 接收命令输出结果。
- 如果返回结果为 Kill 信号的时候，能够重启任务。

通过 GOOS=linux GOARCH=amd64 go build -o climits main.go  命令，编译一个 Linux 版本的可执行程序 `climits`。

### 运行示例

我们已经提前创建了一个叫做 `climits` 的内存相关 CGroup，并且目录下包含 `climits`, `simpleapp` 两个可执行程序。

此时运行命令 `./climits -memory 60`，可以看到如下输出：

``` bash
[root@A04-R08-I197-202-3DGCDB2 climits]# ./climit -memory 60
add pid 48189 to file cgroup.procs
Child pid is 48189
Alloc = 1 MiB	Sys = 66 MiB 
Alloc = 3 MiB	Sys = 66 MiB 
Alloc = 8 MiB	Sys = 68 MiB 
Alloc = 8 MiB	Sys = 68 MiB 
Alloc = 16 MiB	Sys = 68 MiB 
Alloc = 16 MiB	Sys = 68 MiB 
...
Alloc = 32 MiB	Sys = 134 MiB 
Alloc = 32 MiB	Sys = 134 MiB 
cmd return with error: signal: killed
app is killed by system
restart app..
add pid 48256 to file cgroup.procs
Child pid is 48256
Alloc = 1 MiB	Sys = 66 MiB 
Alloc = 3 MiB	Sys = 68 MiB 
Alloc = 4 MiB	Sys = 68 MiB 
Alloc = 12 MiB	Sys = 68 MiB 
^CGot signal: interrupt
```

通过输出可以看出，当内存超过一定限制后，进程 `48189` 会被 Kill 掉，守护程序收到 Kill 信号后，会先关闭老进程，再重启新进程 `48256`。

## 总结

这篇文章主要简单介绍了 CGroup 控制进程内存的原理，并通过 Go 代码实现一个简单的进程守护，支持内存限制和进程重启。我们还可以通过它来查看进程内存使用详细信息，以此完成一个简易内存 container。

参考链接：

- [man7/cgroups](http://man7.org/linux/man-pages/man7/cgroups.7.html)
- [限制cgroup的内存使用](https://segmentfault.com/a/1190000008125359)
- [climits源代码](https://github.com/songjiayang/climits)