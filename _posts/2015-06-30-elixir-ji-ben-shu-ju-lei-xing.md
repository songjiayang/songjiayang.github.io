---
title: "elixir 的基本类型"
date: 2015-06-30 22:31:00 +0800
tags: [elixir]
---

### 基本类型：

elixir 的基本类型包括有 integers, floats, booleans, atoms, strings, lists 和 tuples（元组）， 它们基本形式如下：

```
iex> 1          # integer
iex> 0x1F       # integer
iex> 1.0        # float
iex> true       # boolean
iex> :atom      # atom / symbol
iex> "elixir"   # string
iex> [1, 2, 3]  # list
iex> {1, 2, 3}  # tuple
```

#### 注意 ⚠：
1. `/` 返回是 float，所以 10/5 的结果是 2.0, 如果想得到整数，可以使用 div 函数，如 `div(10,2)`。
2. float 在 elixir 中是 64 位的双精度数。
3. boolean 实质上也是 atom, 所以  is_atom(true) 和 is_boolean(true) 都是 true。
4. elixir 中 string 使用双引号，默认是 UTF-8 编码， 其中 byte_size(str) 用于查看字节数， String.length(str) 用于查看字符串长度。
5. 如果一个 list(链表) 中的数字是可以打印的 ASCII 编码数字，那么 elixir 将其打印成一串字符， 例如 [104, 101, 108, 108, 111] 打印成 'hello'。单引号包着的其实是一串字符列表，所以 [104, 101, 108, 108, 111] == 'hello' 结果为 true 。 length 方法用于查看 list 长度。
6. tuples(元组) 其实就是传统意义上的数组，内存地址连续。可以使用 tuple_size(tuple) 查看 tuple 长度。
7. list 和 tuple 的区别就如同链表和数组的区别；其中他们都可以存储不同类型的元素。
8. 可以使用 is_type 的方式查询数据类型， 例如 is_integer, is_float, is_atom, is_boolean, is_binary, is_list, is_tuple 等。


### 匿名函数：

函数使用关键字 fn 声明， end 结束。匿名函数简单使用：

```
iex> add = fn a, b -> a + b end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> is_function(add)
true
iex> is_function(add, 2)
true
iex> is_function(add, 1)
false
iex> add.(1, 2)
3
```

参考链接：  [http://elixir-lang.org/getting-started/basic-types.html](http://elixir-lang.org/getting-started/basic-types.html) 。
