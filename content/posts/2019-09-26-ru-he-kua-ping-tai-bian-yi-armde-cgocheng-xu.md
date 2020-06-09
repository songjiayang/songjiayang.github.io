---
title: "如何在 Ubuntu 上交叉编译 ARM 架构的 CGO 程序"
date: 2019-09-26 17:38:08+0800
tags: [go]
---

我们都知道在不涉及 CGO 的时候，Go 的交叉编译非常简单，只需要设置对应的 GOOS 和 GOARCH 即可，但当涉及到 CGO 时，问题就变得有点复杂了，因为你需要指定特定的 GCC。

例如，想在 Ubuntu 上交叉编译一个带有 CGO 的动态库，目标 CPU 架构为 arm，我们该如何操作呢？

## 示例代码

```
# shared.go
package main

import "C"

//export Sum
func Sum(a, b int) int {
	return a + b
}

func main(){}
```

这个代码使用到了 CGO，然后暴露一个 `Sum` 方法，实现两个整数相加。

## 编译 32 位的 arm

因为通过设置 `CGO_ENABLED=1` 开启 CGO 编译，执行命令如下：

```
CGO_ENABLED=1 GOOS=linux GOARCH=arm go build -buildmode=c-shared -o share.so 
```

但不幸，命令报错： `gcc: error: unrecognized command line option '-marm'`。

正如一开始我提到，交叉编译 CGO 需要选择特定的 arm 交叉编译工具，而 Ubuntu 上编译 32 位的 arm 可以使用 `gcc-arm-linux-gnueabihf`，安装命令如下：

```bash
sudo apt-get update
sudo apt-get install gcc-arm-linux-gnueabihf
```

安装完成后，指定 `CC` 重新编译：

```bash
 CGO_ENABLED=1 GOOS=linux GOARCH=arm CC=arm-linux-gnueabihf-gcc go build -buildmode=c-shared -o share.so 
```

命令成功运行，此时目录下已经生成了一个叫做 `share.so` 的文件，通过 `file` 命令查看其属性，可以确认确实为 `arm 32` 版本。

```
share.so: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, BuildID[sha1]=7b23579ddcbebdfc8f4b68512859661a45d66996, not stripped
```

## 编译 64 位的 arm

交叉编译的时候，不仅要针对平台选择 GCC，而且还要根据操作系统的位数来选，所以 64 位需要选择不同的 GCC，这里推荐 [gcc-linaro-5.3-2016.02-x86_64_aarch64-linux-gnu.tar.xz](https://releases.linaro.org/components/toolchain/binaries/5.3-2016.02/aarch64-linux-gnu/gcc-linaro-5.3-2016.02-x86_64_aarch64-linux-gnu.tar.xz)。

安装命令:

```
wget https://releases.linaro.org/components/toolchain/binaries/5.3-2016.02/aarch64-linux-gnu/gcc-linaro-5.3-2016.02-x86_64_aarch64-linux-gnu.tar.xz 

tar xvf gcc-linaro-5.3-2016.02-x86_64_aarch64-linux-gnu.tar.xz -C /usr/lib/

echo 'export PATH="$PATH:/usr/lib/gcc-linaro-5.3-2016.02-x86_64_aarch64-linux-gnu/bin"' >> ~/.bashrc

source ~/.bashrc
```

安装完成，重新执行编译命令：

```
 CGO_ENABLED=1 GOOS=linux GOARCH=arm64 CC=aarch64-linux-gnu-gcc-5.3.1 go build -buildmode=c-shared -o share.so 
```

编译成功，并产生一个 share.so 文件，同样我们使用 `file share.so` 查看其元信息为：

```
share.so: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, BuildID[sha1]=5b0e7ff7c3af178039a7b934df8ca3e7572ab5b5, not stripped
```

到目前为止，我们已经成功在 Ubuntu 系统上交叉编译出了 CGO 程序的 arm 和 arm64 两个版本。

## 总结

当 Go 交叉编译涉及到 CGO 时，只指定 GOOS 和 GOARCH 是不够的，还需要通过 CC 参数指定相应的 GCC 版本，而 GCC 的选择又与当前系统以及目标架构有关：

- 交叉编译目标 CPU 架构（包括 32位 还是 64位）。
- 交叉编译所在操作系统。