---
title: "benchmarks in go"
date: 2016-06-26 22:31:00 +0800
tags: [go]
---

![benchmark-in-go.png](/images/benchmark-in-go.png)

在涉及代码性能优化的时候，benchmarks 自然少不了，那么如何在 golang 中做 benchmarking 呢？

这里我们可以使用 `testing` 包，它包含了 benchmark 全部代码。

### 1. 定义 benchmark 用例

与单元测试相似，benchmark 用例的定义也是靠函数名来约定，只不过函数名从 `Testxx` 换成了 `Benchmarkxx`.

 例如:

```

func BenchmarkHello(b *testing.B) {
   for i := 0; i < b.N; i++ {
       fmt.Sprintf("hello")
   }
}
```

### 2. 运行 benchmark 用例

运行 benchmark 需要使用 go test 命令，相关参数有:

```
-bench regexp
    Run benchmarks matching the regular expression.
    By default, no benchmarks run. To run all benchmarks,
    use '-bench .' or '-bench=.'.

-benchmem
    Print memory allocation statistics for benchmarks.

-benchtime t
    Run enough iterations of each benchmark to take t, specified
    as a time.Duration (for example, -benchtime 1h30s).
    The default is 1 second (1s).
```

例如我可以运行 `go test -bench=BenchmarkHello -benchmem -benchtime=10s` 来执行测试用例：
```
BenchmarkHello-4	100000000	       124 ns/op	       5 B/op	       1 allocs/op
```
