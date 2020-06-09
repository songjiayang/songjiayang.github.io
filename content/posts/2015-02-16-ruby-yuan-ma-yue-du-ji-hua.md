---
title: "一起阅读 Ruby 类库的源码吧！"
date: 2015-02-16 22:31:00 +0800
tags: [ruby]
---

![I want you](https://drscdn.500px.org/photo/23171313/m=1170/c150b74f80ce6008772339fb30b2c9c8)
<span class='center'>(I want you: 图片来源 500px)</span>

事情源于一天和同事讨论 Ruby 单例实现，当时为了了解 `Singleton` 这个 module 更多东西 ，顺手将 Ruby
代码 clone 下来，结果惊奇发现其代码注释非常详细，十分方便阅读：

![singleton-code](http://songjiayang.qiniudn.com/singleton-code.png)

所以就有了这个阅读计划。
#### 开始阅读

1. 克隆最新的ruby代码， git clone git@github.com:ruby/ruby.git 。
2. 需要阅读的代码在 ruby_xx/lib 文件夹里面。
3. ok, 开始吧，先从自己最熟悉的 module 下手。

#### 结束时间

现在我正阅读的版本是 2.2, 需要阅读的文件总计 `find . -name "*.rb" | wc -l`（626个）。
这意味着，即使每天阅读一个模块，几乎都要花费半年时间了，可见这是一件长期的计划。

阅读源码是一种好的习惯，也是持续学习的过程，而且好的代码总能让人反复咀嚼，因为此计划暂无 deadline。

#### 阅读产物

阅读会根据自己理解，不定期通过博客给出有一些代码演示或旧代码重构，你期待了吗？如果你感兴趣，加入此计划吧。
