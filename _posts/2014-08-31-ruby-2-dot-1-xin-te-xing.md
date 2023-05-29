---
title: Ruby 2.1 新特性
date: 2014-08-31 23:31:00 +0800
tags: [ruby]
---

原文链接 [Ruby 2.1](http://rkh.im/ruby-2.1)。

昨天 Ruby 2.1 第一个预览版 [发布了](https://www.ruby-lang.org/en/news/2013/09/23/ruby-2-1-0-preview1-is-released/)。 发布注意事项已经给出了所有的变化，但是它并没有详细的描述。如果想一一搞明白这些改变是什么意思的话，你可以尝试花费一些时间在 Ruby 问题跟踪列表里面。既然我已经知道了大部分的改变，我想我有必要在这里写个简要。  

> 很显然， Ruby 2.1 包含了 Ruby 2.0 所有的新特性，所以，2.0 的特性在这里就不一一赘述。

#### Refinements

Refinements 其实早在 Ruby 2.0 就已经有了，但是当时由于它引发了比较大的争议，所以他们减少了功能，添加了警告，并将这个特性标记为实验性质。

Refinements 允许你的猴子补丁只作用于单个 Ruby 文件:

```
module Foo
  refine String do
    def foo
      self + "foo"
    end
  end
end

using Foo
puts "bar".foo
```

超出了这个文件作用范围， `String` 将不再有 `foo` 这个方法了。

Ruby 2.1， Refinements 不再是实验性的了，并且也不会产生警告了, 另外还有几个小的调整使他变得更可用了。

```
module Foo
  refine String do
    def foo
      self + "foo"
    end
  end
end

module Bar
  using Foo
  puts "bar".foo
end
```

请注意, 过度使用 refinements 会导致代码混乱，Ruby 的开发者们都明确说过他们也没有办法解决这个问题，所以切忌滥用。

#### 十进制精度计算

你可能已经知道，到目前为止，完全保证十进制小数计算结果的精度还是一种梦想。

```
irb(main):001:0> 0.1 * 3
=> 0.30000000000000004
```

许多 Ruby 开发者们使用整数来解决上述问题，但是你不要忘了，这种方法只适用于小数位数固定的情况，假如小数精度无法固定的话， 你将不得不使用有理数来处理了。有理数并不糟糕，只是他们没有一个漂亮的语法罢了（简短的改变在返回Integer进行#/的数值的时候）

Ruby 2.1 提供了 `r` 这个后缀来解决十进制有理数精度固定计算的问题。

```
irb(main):001:0> 0.1r
=> (1/10)
irb(main):002:0> 0.1r * 3
=> (3/10)
```

#### 字符串#freeze优化

当你的代码中有一个字符串，程序每次运行到这一行的时候， Ruby 都将创建一个新的字符串。 这很可能发生， 因为字符串在 Ruby 中是可变的。 这就是为什么 `symbols` 在很多情况下更高效的缘故。然而， `symbols` 并不是字符串。 例如，如果你想对用户的输入和一个 `symbol` 进行比较 ， 你要么必须将 `symbol` 转化为字符串，或者将字符串转化为 `symbol`。这意味着你要么开打了自己的拒绝服务攻击，而且 `symbol` 是不被系统垃圾回收的，要么你又添加了一个额外的字符串。

使用 `symbols` 的字符串的插值问题还是会创建一个新的字符串，同样如此，往socket里面写一个 `symbol` 也是一样的结果，这样的例子数不胜数。

解决这个问题的办法就是将字符串存储在一个常量里面，并且以后都使用常量来代替。

```
class Foo
  BAR = "bar"

  def bar?(input)
    input == BAR
  end
end
```

 现在，为了解决字符串可变的问题，很多时候，你不得不冻结这个字符串(虽然这个可以防止改变，但是它并没有带来多少性能上的提升)。

```
 class Foo
  BAR = "bar".freeze

  def bar?(input)
    input == BAR
  end
end
```

这个变得有点乏味了。 幸运的是，Ruby 2.1 提供了一个语法来处理这种事情：

```
class Foo
  def bar?(input)
    input == "bar"f
  end
end
```

它将创建一个冻结的字符串，并且会重复使用，无论代码执行多少次。

如果你的代码品味和我差不多的话，你可能会觉得这个语法看起来太奇怪，太像上面提到的十进制数的语法了。 我发现这里有一个绝招，可以让它看起来更舒服：

```
class Foo
  def bar?(input)
    input == %q{bar}f
  end
end
```

这里有一个 [issue](https://bugs.ruby-lang.org/issues/8909) 建议将这个语法扩展到数组和 Hash 里面去。我更加喜欢 [Charlie Somerville](https://charlie.bz/) 提交的补丁和建议的语法。

#### 必须的关键字参数

由于某种原因，它并没有在发布公告里面被提到。

Ruby 2.0 是这样介绍关键字参数：

```
def foo(a: 10)
  puts a
end

foo(a: 20) # 20
foo        # 10
```

然而， 这种方式的关键字参数需要一个默认值。 在 Ruby 2.1 中，你现在可以这样使用关键字参数：

```
def foo(a:)
  puts a
end

foo(a: 20) # 20
foo        # ArgumentError: missing keyword: a
```

#### 方法定义将返回方法的名字

在以前的 Ruby 版本中， 通过 `def` 定义方法将返回 `nil`:

```
def foo ; end # => nil
```

在 Ruby 2.1 中， 它将以 `symbol` 的形式返回方法名字：

```
def foo ; end # => :foo
```

这个在元编程或者写其他整洁代码中非常有用。例如， 你是否知道 `private` 方法也可以传入参数呢？

```
# 只有foo这个方法是私有的
class Foo
  def foo
  end

  private :foo

  # bar 这个方法并不受影响
  def bar
  end
end
```

我在 [mustermann](https://github.com/rkh/mustermann) 中大量使用了这种方式。

现在， `def` 返回方法的名字，这就意味着我们可以使一个实例方法私有:

```
# 只有 foo 和 bar 这两个方法是私有
class Foo
  private def foo
  end

  private \
  def bar
  end

  def baz
  end
end
```

我已经创建了一个[问题](https://bugs.ruby-lang.org/issues/8947)，扩大这一行为到其他地方。

#### 从字符串中删除垃圾字符

Ruby 现在提供了非常方便从字符串中删除垃圾字符的方法：

```
some_string.scrub("")
```

这完全有必要让 Ruby 自己处理这个问题，我曾经为了解决这个问题，还专门写了一个[库](https://github.com/rkh/coder).

#### 字符扫描支持命名捕获

标准库中的字符扫描真好用。事实上， Rails 用它来解析路由的参数， Sinatra 2.0 也是如此。可以参考 Aaron's tutorial 的教程 [practicingruby.com](https://practicingruby.com/articles/parsing-json-the-hard-way?u=90296723ac) 。

Ruby 1.9 中介绍了在正则表达式中进行命名捕获，但是字符扫描还不支持：

```
require 'strscan'
s = StringScanner.new("foo")
s.scan(/(?<bar>.*)/)
puts s[:bar]
```
在 Ruby 2.0 上执行结果是：  

```
TypeError: no implicit conversion of Symbol into Integer
```

Ruby 2.1 是：

```
foo
```

实际上这个补丁是我提交的,（我以前用的 `goto`, 我的天啊）

#### 访问网络接口

现在你可以通过 `Socket.getifaddrs:` 访问网络接口了:

```
require 'socket'

Socket.getifaddrs.each do |i|
  puts "#{i.name}: #{i.addr.ip_address}" if i.addr.ip?
end
```

运行以上代码，在我的电脑上打印结果：

```
lo0: fe80::1%lo0
lo0: 127.0.0.1
lo0: ::1
en0: fe80::1240:f3ff:fe7e:594e%en0
en0: 192.168.178.30
en2: fe80::3e07:54ff:fe6f:147a%en2
```

#### 更快数字运算

Ruby 2.1 现在在大数字计算的方面，速度更快了，因为它的大整数扩展到了128位，而且还使用GNU的多进度运算库得到额外的性能提升。

#### 虚拟机调整

底层虚拟机不再使用全局方法缓存, 取而代之的是细粒度方法缓存 , 如果你对这个感兴趣，可以看看我曾经的[一个演讲](https://speakerdeck.com/rkh/aloha-ruby-conf-2012-message-in-a-bottle)。

#### RGenGC

Ruby 2.1添加了一个“受限的”分代收集器, 标记时间更短，大大减少了垃圾回收的耗时。

如果还是像以前那样， 这是很难更改的，因为Ruby的内部/外部C API的缘故。然而， 在 Ruby 2.1 中，将对像分为受写屏障保护的对象和非保护的对象，只有受保护对象能够被提升为老生代，（这一点在RGenGC中是严格限定的）。Koichi Sasada 在 [RubyKaigi](http://rubykaigi.org/2013/talk/S73) 和 [Euruko](http://www.ustream.tv/recorded/35107339/highlight/377033) 有一个关于这个的讨论.

#### 升级RubyGems

RubyGems 已经升级到 2.2.0， 它带来了 [一些小的改变](https://github.com/rubygems/rubygems/blob/master/History.txt) 。

#### 仅仅是个预览

请记住，昨天发布的版本只是个预览版，以上提到的只是一些改变。

<hr>

参考资料：

1. [http://www.oschina.net/translate/ruby-2-1-in-detail](http://www.oschina.net/translate/ruby-2-1-in-detail)
2. [http://www.oschina.net/translate/ruby21-rgengc](http://www.oschina.net/translate/ruby21-rgengc)
