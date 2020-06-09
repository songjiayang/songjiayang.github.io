---
title: "Ruby Proc 迷思"
date: 2016-03-07 22:31:00 +0800
tags: [ruby]
---
![proc_lambda.png](/images/proc/proc_lambda.png)

一谈到闭包 (`Proc`)， Rubyist 的脑袋里往往会浮现出 blocks, Proc, lambda 之类概念。

此时，你心中可能开始嘀咕： why ? 这概念是不是太多了， Ruby 的闭包怎么这么复杂啊。

其实不然，Ruby 的闭包并没想象的那么复杂，我们需要通过现象看本质，揭开它的神秘面纱。

### 一、了解 ruby 闭包创建方式

#### 1. `blocks`
![proc_1.png](/images/proc/proc_1.png)

#### 2. `proc`
![proc_6.png](/images/proc/proc_6.png)

#### 3. `Proc.new`
![proc_2.png](/images/proc/proc_2.png)

#### 4. `lambda`
![proc_3.png](/images/proc/proc_3.png)

#### 5. `->`
![proc_4.png](/images/proc/proc_4.png)

#### 6. method to `to_proc`
![proc_5.png](/images/proc/proc_5.png)

好了，以上就是在 Ruby 中创建闭包的不同方式，通过运行结果你不难发现：

1. 所有方式返回结果都是 Proc 的对象（实例）。
2. Proc 对象有两种： lambda 和非 lambda。 我们可以通过  `lambda?` 方法来检查。

### 二、不同方法特别说明

1. `blocks` 是我们平时写的最多的，主要配合 `yield` 使用，这种方式有个问题是没法共用 block 。
2. `Proc.new` 就是对象实例方法。
3. `proc` 和 `lambda` 是 Ruby `Kernel` 模块提供的方法，用于生成一般 proc 和 lambda proc 。
4. `->` 是 Ruby 操作符，用于生成 lambda proc 。
5. `to_proc` 是 Ruby `Method`对象的实例方法， 用于将已有 method 转变成 lambda proc 。

### 三、 一般 proc 和 lambda proc 区别

1. lambda 会对参数个数验证，proc 不会验证。
2. lambda 会执行 return, proc 遇到 return 会中断。

总结： 无论 proc 还是 lambda，其实本质都是一个东西，因为我们最终用到的都是 Proc 的对象，所以闭包就是 Proc 对象。
