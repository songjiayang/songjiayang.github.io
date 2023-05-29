---
title: "一篇帖子引发的思考"
date: 2015-01-16 22:31:00 +0800
tags: [ruby]
---

这是一篇在 Ruby－China 上讨论的比较激烈的帖子，原帖地址 [https://ruby-china.org/topics/23766](https://ruby-china.org/topics/23766)。

原帖属于楼主的经验分享帖，一个关于 sidekiq 的 worker 被全部占用，无法释放的问题。
出现这个问题的原因是，楼主 sidekiq worker 中有一段网络请求的代码，但由于网络请求服务不稳定，导致无响应或者响应很慢，所以，worker 全部都被占用了。

解决这个问题的办法很简单，就是设置 Timeout 时间。

> IO 请求都需要设置超时时间。

楼主在原帖中还提到了 sidekiq 常见 Frozen worker 的两种情况：

1. DNS lookup - resolving a hostname might hang. This has a serious side effect in MRI of locking up everything because of the way MRI uses DNS by default. The solution is to run require 'resolv-replace' in your initializer, which installs a pure Ruby DNS resolver that works concurrently.
2. Net::HTTP - unresponsive remote servers can cause a Net::HTTP call to hang and lock up your workers. Set open_timeout to ensure your code raises an exception rather than hanging forever.

楼主的问题属于第2点，但是第1点却更引起我的兴趣，因为它提到了 MRI 的 [GIL](http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil) 。

假设有线程a,b，其中a 调用了一个会被 block的 C 扩展，那么 b 线程也会被 block 吗 ？

第一步： a 线程未调用 C 扩展，执行如下代码：

```
# test.rb

a = Thread.new do
  while true do
    puts "aaa"
  end
end

b = Thread.new do
  while true do
    puts "bbb"
  end
end

sleep

```

输出结果：`bbb aaa` ,  可见， a,b 线程都执行了。


第二步：编写一个会被block的 C 扩展（死循环）：

```
# 一个会被 block的 C 扩展程序

#include "ruby.h"

VALUE BlockExt = Qnil;
VALUE perform(VALUE self);

void Init_block_ext() {
  BlockExt = rb_define_class("BlockExt", 0);
  rb_define_singleton_method(BlockExt, "run", perform, 0);
}

VALUE perform(VALUE self) {
  while(Qtrue){  }
  return Qnil;
}

```

第三步： a 线程调用 C 扩展：

```
# test.rb
require_relative 'ext/block_ext'
a = Thread.new { BlockExt.run }
b = Thread.new do
  while true do
    puts "bbb"
  end
end

sleep
```

运行 `ruby test.rb`， 你会发现 b 线程也会 block。

出现这个问题的原因是因为 a 线程调用了外部扩展程序，由于 Ruby GIL 的缘故，
同一时间只能有一个线程(a) active, 而由于外部程序 block住了，导致无返回，
从而a程序一直处于active状态，所以b线程一直处于 unactive状态，也就被block住了。

结论：Ruby 多线程都需要注意一些 C Extensions 的 GIL block 住整个进程的问题，自己编写的 C 扩展，一定要确保返回和结束。

代码地址： https://github.com/songjiayang/rc-ext-gil
