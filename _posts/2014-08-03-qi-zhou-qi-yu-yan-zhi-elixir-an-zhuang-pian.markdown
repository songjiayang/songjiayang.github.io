---
title: 七周七语言之 elixir 初级篇
date: 2014-08-03 20:19:04 +0800
tags: [elixir]
---

今天听闻ElixirConf 2014 slide出来了，关于报道请移步 [RubyChina讨论帖子](https://ruby-china.org/topics/20809)。

看到这个消息心中不免有些小激动，因为它的语法终于快稳定下来了，版本1.0的发布也是紧接的事情，于是决定尝尝鲜。

### Elixir初级篇

>>测试环境： Mac OS 10.9.4

#### 1.1 安装Erlang 

由于Elixir是一门运行在erlng虚拟机上的函数式、面向并行的编程语言，自然需要erlang运行环境。

最新的elixir(v0.14.0)需要erlang17.0或者更新。

使用[Precompiled packages](https://www.erlang-solutions.com/downloads/download-erlang-otp) 安装Erlang很容易，*只是国内下载网络太慢*。
如果你觉得以上链接下载太慢的话，可以尝试[此链接](http://pan.baidu.com/s/1o6qgdN8)。

我下载的软件包是otp_src_17.1.tar.gz。

进入otp_src_17.1.tar.gz下载根目录，解压erlang软件包，编译，安装 。

```
 tar -vxzf otp_src_17.1.tar.gz
 cd otp_src_17.1
 ./configure
 make
 make install
```

你可以在Terminal中运行 `erl`来检查Erlang的版本,你将看到类似信息：

```
Erlang/OTP 17 [erts-6.1] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]
```

#### 1.2 elixir安装

由于我的系统是OS，为了不折腾所以采用homebrew安装。

```
brew update
brew intall elixir
```

当安装结束后，你可以在Terminal中运行 `iex`进入elixir的交互模式，将看到以下信息：

```
Erlang/OTP 17 [erts-6.1] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]
Interactive Elixir (0.15.0) - press Ctrl+C to exit (type h() ENTER for help)
```

其他系统使用源的安装方式以及预编译安装方式，请参考[官网](http://elixir-lang.org/getting_started/1.html#1.2-distributions)

#### 1.3 elixir初体验

进入`iex`

```
Interactive Elixir (0.15.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> 40+2
42
iex(2)> 40+1.3
41.3
iex(3)> IO.puts "hello world"
hello world
:ok
iex(4)> "hello" <> " world"
"hello world"
iex(5)> defmodule Hello do
...(5)>   def world do
...(5)>     IO.puts "Hello World"
...(5)>   end
...(5)> end
{:module, Hello,
 <<70, 79, 82, 49, 0, 0, 4, 208, 66, 69, 65, 77, 69, 120, 68, 99, 0, 0, 0, 94, 131, 104, 2, 100, 0, 14, 101, 108, 105, 120, 105, 114, 95, 100, 111, 99, 115, 95, 118, 49, 108, 0, 0, 0, 2, 104, 2, ...>>,
 {:world, 0}}
iex(6)> Hello.world
Hello World
:ok
```

以上测试包含了整数相加，整数和浮点数相加，打印输出，字符串拼接，模块定义和方法的定义/调用。  
值得注意的是：  

1. 在elixir中的字符串拼接使用的是 `<>` 而不是`+`。  
2. 方法的定义必须在`module`中。  

总结:  
1. Elixir安装容易，它的语法也和ruby有几分相识之处（毕竟就是搞ruby那帮人写的嘛）。  
2. 它的出现感觉就是在语言高性能和语法优雅中寻找平衡。只是目前语言还在逐步完善中，1.0版本发布，将是一个里程碑。   
3. 社区也还不够强大，不过对于才出现1年多点elixir而言，它有足够的时间去成长。
