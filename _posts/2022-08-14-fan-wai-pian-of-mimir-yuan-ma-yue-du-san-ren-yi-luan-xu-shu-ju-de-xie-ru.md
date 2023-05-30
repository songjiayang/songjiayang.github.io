---
title: "番外篇 of 《Mimir 源码阅读（三）： 任意乱序数据的写入》"
subtitle: 
date: 2022-08-14  23:39:20+0800
# thumbnail-img: 
# cover-img: 
# share-img: 
tags: [grafana, mimir]
---

我们在上一篇文章[Mimir 源码阅读（三）： 任意乱序数据的写入](/)提到了prometheus的ooo问题。有小伙伴私信再介绍一下ooo的细节，我们出了本次的番外篇。

prometheus对于实时数据处理表现出强大的能力。但是对于历史数据的处理如何，本文通过两个场景，进行一些尝试：

- 时间乱序
- 历史数据

## 测试环境准备

下载最新prometheus版本。找到位于prometheus\tsdb\blockwriter_test.go的测试用例TestBlockWriter()函数。


## 时间乱序

构造一个metric，在不同时间点 1649575406000（2022-4-10 15:23:26）和1649575380000（2022-4-10 15:23:24）时刻的写入数据，各写入1个sample数据

```go
ts1, v1 := int64(1649575406000), float64(7)
ts11, v1 := int64(1649575380000), float64(1)
_, err = app.Append(0, labels.Labels{{Name: "a", Value: "b"}}, ts1, v1)
require.NoError(t, app.Commit())
_, err11 := app.Append(0, labels.Labels{{Name: "a", Value: "b"}}, ts11, v1)
require.NoError(t, err)
require.NoError(t, err11)
```

执行test,会报错

```
c.maxTime: 1649575406000 cur time: 1649575404000
--- FAIL: TestBlockWriter (0.01s)
    d:\opensource\myprometheus\prometheus\tsdb\blockwriter_test.go:51: 
          Error Trace:  blockwriter_test.go:51
          Error:        Received unexpected error:
                        out of order sample
          Test:         TestBlockWriter
```

可以看到，tsdb报错了一个“out of order sample”的错误。我们跟踪一下代码，

```go

// appendable checks whether the given sample is valid for appending to the series.
func (s *memSeries) appendable(t int64, v float64) error {
  c := s.head()
  if c == nil {
    return nil
  }
  if t > c.maxTime {
    return nil
  }
  if t < c.maxTime {
    fmt.Println("c.maxTime:", c.maxTime, "cur time:", t)
    return storage.ErrOutOfOrderSample
  }
```

可以看到，由于用例的ts11(1649575380000)时间比c.maxTime(1649575406000 )小，所以报乱序的错误。意思就是，最后到达的时间ts11，比最先到达的时间ts1，还要小，系统不会处理。即，系统只会处理时间点向后流逝的数据。 

实际上，prometheus对于时间乱序的metric，会在两处进行截断，

1）app.Append里面，对metric进行内存记录时，如上述场景，会进行数据丢弃。
2）app.Commit里面，对内存中的metrics进行汇聚时，会再次进行乱序校验。

```go
func (a *headAppender) Commit() (err error) {
  ...
  series.append(s.T, s.V, a.appendID, a.head.chunkDiskMapper)
   => func (s *memSeries) append (...){
       ...
      // Out of order sample.
      if c.maxTime >= t {
        return false, chunkCreated
      }
    }
 }
```

对短时间数据上报时，出现乱序的场景。这种错误，在实时监控中也可能存在，如果前端push消息到Kafaka，然后被不同的消费队列进行处理时，最终到达tsdb后端存储时，就会存在偶然的乱序。

需要说明的是，out of order 针对的是某一个时间线series即将append的采样点，和当前chunk记录的最新时间比较的结果。

## 历史数据

在实际IoT使用中，也会存在一种场景：今天上报了一批时序数据，但发现前几小时/天的离线数据，需要补录。对于这种场景，我们继续观察prometheus什么表现。

构造一个metric，在不同时间点 1649575406000（2022-4-10 15:23:26）和1649564604000（2022-4-10 12:23:26）时刻的写入数据，各写入1条sample数据。

```go
ts1, v1 := int64(1649575406), float64(7)
ts11, v11 := int64(1643990400), float64(1)
_, err = app.Append(0, labels.Labels{{Name: "a", Value: "b"}}, ts1, v1)
//require.NoError(t, app.Commit())
_, err11 := app.Append(0, labels.Labels{{Name: "a", Value: "b"}}, ts11, v11)
require.NoError(t, err)
require.NoError(t, err11)
//ts2, v2 := int64(55), float64(12)
/*ts2, v2 := int64(1649297575), float64(12)
_, err = app.Append(0, labels.Labels{{Name: "c", Value: "d"}}, ts2, v2)
require.NoError(t, err)*/
require.NoError(t, app.Commit())
```

报错如下：

```go
h.minValidTime.Load(): -9223372036854775808 h.MaxTime() 1649575406000 h.chunkRange.Load(): 3600000
a.minValidTime: 1649571806000 cur time: 1649575406000
a.minValidTime: 1649571806000 cur time: 1649564604000
--- FAIL: TestBlockWriter (0.01s)
    d:\opensource\myprometheus\prometheus\tsdb\blockwriter_test.go:51: 
          Error Trace:  blockwriter_test.go:51
          Error:        Received unexpected error:
                        out of bounds
          Test:         TestBlockWriter
```

报错来自代码的a.minValidTime：

```
func (h *Head) appender() *headAppender {
  ...
  fmt.Println("minValidTime:", h.appendableMinValidTime())
  return &headAppender{
    head:                  h,
    minValidTime:          h.appendableMinValidTime(),
    ...
  
func (h *Head) appendableMinValidTime() int64 {
  // Setting the minimum valid time to whichever is greater, the head min valid time or the compaction window,
  // ensures that no samples will be added within the compaction window to avoid races.
  fmt.Println("h.minValidTime.Load():", h.minValidTime.Load(), "h.MaxTime()", h.MaxTime(), "h.chunkRange.Load():", h.chunkRange.Load()/2)
  return max(h.minValidTime.Load(), h.MaxTime()-h.chunkRange.Load()/2)
}
```

可以看到，影响minValidTime的参数，就是h.MaxTime()-h.chunkRange.Load()/2，结合实例，maxtime就是1649571806000,chunkrange默认是2h。所以历史数据的最小值，不能小于 1649571806000-7200/2=1649571806000 (2022-4-10 14:23:26) 。这个时间是prometheus落盘的默认chunkRange分片时长2h的一半,即1h。这是考虑到，tsdb数据压缩默认2h对齐，为了减少2h时间附近数据点的影响。综上，对于历史数据的时间戳，不能小于落盘的block时间分片长度的一半。

需要说明的是，out of bound针对的是某一个租户（tsdb）即将append的采样点，和当前chunk记录的最小有效时间比较的结果。一个极端的场景就是：对于一个租户，大部分的series都是跟着时间刷新的，但是如果有一个series最新写入的一条sample数据，也是1h以前的，就会出现out of bound的错误。

## 总结

通过上述举例，我们看到，对于时序数据的历史数据处理时，prometheus当前不具备处理上述两种场景的能力，会出现：out of order 和 out of bound 的错误。对于IoT场景，经常存在弱网环境，或者历史数据导入的场景，如果处理这种离线数据时，就需要克服上述1和2的场景。