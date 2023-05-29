---
title: "如何使用正则格式化 alertmanger 的告警信息"
subtitle: GitHub 一个提问解答，Alertmanager 告警消息 format。
date: 2021-04-18 20:43:38+0800
# cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
tags: [prometheus]
---

你可能会好奇，为什么会讨论这个话题？

这是因为前段时间有个朋友在 GitHub 上提问，问题是 Wechat 告警模板中隐藏 instance 中 IP 前几位该如何实现？

## 朋友的思路

他的思路是使用 Go 的 strings 包，先对 instance IP 按照 . 分割，然后再将 ["***", str[3], ".", str[4]] 进行拼接。

故一开始告警模版修改为：

<code>
import (
  "strings"
)

告警机器：{{ strings.Join("***",strings.Split(.Labels.instance, ".")[3])}} {{ .Labels.device }}
</code>

但当 Alertmanager 在解析的时候报错了，错误信息为 `failed to reload config: template: wechat.tmpl:4: function "strings" not defined`。

这是因为 Alertmanager 使用的是 Go 的 html/tempalte 包来做模版解析，当我们想引入其它包或者自定义函数，应该在解析模版之前，通过 Funcs 函数来进行注册，从而才能在模版中使用该注册函数。

## 朋友的解法

其实他很快也发现了不能直接在告警模版中引入其它 Go 包，但他的解法是克隆一份 alertmanger 代码，在 template/template.go#L127 的 DefaultFuncs 这个 Map 中注册了一个叫做 strSplitAndJoin 的自定义的函数。

函数具体内容如下：

```
var DefaultFuncs = FuncMap {
    ....
  "strSplitAndJoin": func(s string) string {
        arr := strings.Split(s, ".")
        ss := []string{"***", arr[2], ".", arr[3]}
        return strings.Join(ss, "")
    },
}
```

最后再将模版对应位置修改为：

```
告警机器：{{ strSplitAndJoin .Labels.instance }} {{ .Labels.device }}
```

虽然朋友的方法能够满足需求，但这样做难道就是最佳方案吗？

个人感觉这样做有两个小问题：

- 克隆了代码，增加了维护的成本。
- strSplitAndJoin 函数实现并没有那么通用，输出的内容格式较为固定。

## 更优雅的解法

其实朋友在添加自定义函数的时候，应该也会看到 Alertmanger 已经给我们注册好了一些较为通用的函数

```
var DefaultFuncs = FuncMap{
  "toUpper": strings.ToUpper,
  "toLower": strings.ToLower,
  "title":   strings.Title,
  // join is equal to strings.Join but inverts the argument order
  // for easier pipelining in templates.
  "join": func(sep string, s []string) string {
    return strings.Join(s, sep)
  },
  "match": regexp.MatchString,
  "safeHtml": func(text string) tmplhtml.HTML {
    return tmplhtml.HTML(text)
  },
  "reReplaceAll": func(pattern, repl, text string) string {
    re := regexp.MustCompile(pattern)
    return re.ReplaceAllString(text, repl)
  },
  "stringSlice": func(s ...string) []string {
    return s
  },
}
```

而朋友的这个需求，其实可以直接使用 reReplaceAll 来实现，本质无非就是将一个 IP 地址的前几位进行正则替换。

所以更为简单的配置模版可以为：

{% raw %}
告警机器：{{ reReplaceAll  "([0-9]{1,3}\\.){2}" "***" .Labels.instance }} {{ .Labels.device }}
{% endraw %}

修改完模版自测能够工作，并且输出结果和朋友添加的 strSplitAndJoin 效果一致。

## 总结

Alertmanger 在进行模版渲染的时候，已经帮我们注册好了一些较为通用的函数，基本能够满足我们日常格式化输出的需求。但如果你的需求较为复杂，也可以像这位朋友一样，克隆一份代码，注册一些独特的函数即可。