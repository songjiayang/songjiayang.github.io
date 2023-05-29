---
title: "A bug about prometheus rule incorrect configuration"
date: 2016-11-02 22:31:00 +0800
tags: [prometheus]
---

随着对 Prometheus Alertmanager 深入了解，我们配置的告警通知已非常具有可读性。

但是有一个问题不断困扰着我们，问题现象大致为，在逻辑上我们认为是同一条的告警内容，在告警通知里面出现多次，而且已经过时告警还在不断警告（一直报）。

我们可以将这个问题简单总结为两点:  
1. Prometheus rule 一条查询结果，在 Alertmanager 却对应了多条。  
2. Alertmanager 里面有警报一直处于未恢复的状态（没有收到 Prometheus 发送的 resolved 请求）。

一开始怀疑，是不是 Prometheus 向 Alertmanager 发送的 resolved 通知有丢失, 导致部分历史警告一直处于未恢复状态。

但是排查，发现 Prometheus 和 Alertmanager 在这段时间都没有被重启过。即使有类似现象也是低频的，肯定不会出现如此多的过期告警，而且这种假设仅能部分解释上述问题的第 2 点。

看来要彻底搞清楚这个 bug，必须先弄清楚 Prometheus 中一条告警是怎么和 Alertmanager 中的告警通知关联的。

### 1. 如何判断两条告警其实是同一告警

根据 Prometheus `common/alert` 中的代码可知，所有具有相同 Labels 的告警为同一告警。

那如何才算具有相同 Labels 呢，它的计算逻辑为对一个 map 按照 key 排序，然后依次对 key, value 取值，做 hash，代码逻辑大致为：

```golang
func labelSetToFastFingerprint(ls LabelSet) Fingerprint {
	if len(ls) == 0 {
		return Fingerprint(emptyLabelSignature)
	}

	var result uint64
	for labelName, labelValue := range ls {
		sum := hashNew()
		sum = hashAdd(sum, string(labelName))
		sum = hashAddByte(sum, SeparatorByte)
		sum = hashAdd(sum, string(labelValue))
		result ^= sum
	}
	return Fingerprint(result)
}
```

详情请[参考](https://github.com/prometheus/common/blob/9a94032291f2192936512bab367bc45e77990d6a/model/signature.go#L80) 。

### 2. Prometheus 产生一条 Alert 的逻辑

Prometheus 执行配置中的 rule 查询条件，如果查询结果不为空，那么就会产生告警，而告警条数与查询出的纪录条数一致。

当 rule 查询出结果以后，Prometheus 会创建对应的的告警，而告警信息的 Labels 中除了 Prometheus 自动加入的 `alertname`，还有自己 rule 定义的 Labels。

这条告警内容会被 Prometheus 缓存起来，它缓存的逻辑大致为 alertname => PormQL 查询结果的 Labels 的 hash => 根据 rule 配置产生的告警内容。

### 3. Alertmanager 是如何识别是否为相同告警的呢？

对于 Alertmanager 而言，它收到的告警是打平了的。

因为每个告警的 Labels 里面包含了 rule 的 alertname，区别是否是相同的告警，对 Alertmanager 而言，只需查看是否具有相同的 Labels 即可。 Alertmanager 中的代码大致为：

```golang
// Put adds the given alert to the set.
func (a *Alerts) Put(alerts ...*types.Alert) error {
	a.mtx.Lock()
	defer a.mtx.Unlock()

	for _, alert := range alerts {
		fp := alert.Fingerprint()

		if old, ok := a.alerts[fp]; ok {
			// Merge alerts if there is an overlap in activity range.
			if (alert.EndsAt.After(old.StartsAt) && alert.EndsAt.Before(old.EndsAt)) ||
				(alert.StartsAt.After(old.StartsAt) && alert.StartsAt.Before(old.EndsAt)) {
				alert = old.Merge(alert)
			}
		}

		a.alerts[fp] = alert

		for _, ch := range a.listeners {
			ch <- alert
		}
	}

	return nil
}
```

现在我们已搞清楚了 Prometheus 产生告警和 Alertmanager 中判断相同告警的逻辑。

是时候 review 了一下我们监控系统中 rules 配置了，果然被我发现了一些端倪。我们很多业务类监控的写法类似这样:

```
ALERT xxxx
  IF sum(xxxx) > 1
  FOR 10s
  LABELS {
    value={{$value}},
  }
  ANNOTATIONS {
    summary = "xxxx",
    description = "xxxxx"
  }
```

可以看出，这条 rule 自定义了一个 value 的 label, 它的值是创建告警的时候的查询结果（它是一个变化值），而 Alertmanager 接收到的告警 Labels 大致为:

```
｛ alertname: "xxxx", value: {{$current_value}}, xxx ｝
```

因为 `value` 是一个动态变化的值，所以当 Prometheus 里面缓存的告警内容丢失，就会产生新的告警内容，新的告警会具有不同的 Labels。而当 Alertmanager 接收到这条新的告警的时候，在老的告警里面，很难找到具有相同 Labels 的记录，所以也会对应创建一条新的纪录。

那什么情况会导致 Prometheus 里面缓存的告警内容丢失呢？

1. Prometheus 重启。
2. Rule 配置修改。

由于最近我们在不断调整 rule 配置文件， 看来这个问题确实因它而起。

在弄清楚问题来龙去脉以后，解决起来自然简单很多：

1. 检查 rule 配置文件，删除 value 参数，重新加在 Prometheus 配置。
2. 删除 Alertmanager 的历史数据。

又是一次有趣的 bug 探索，让我对 Prometheus 里面告警生成有了更深认识。 
