---
title: "Prometheus with hot reload"
date: 2016-09-19 22:31:00 +0800
tags: [prometheus]
---

当 Prometheus 有配置文件修改，我们想加载新的配置信息而不停掉服务的时候，可以采用 Prometheus 提供的热更新的方法。

热更新的加载方法有两种：

1. kill -HUP pid
2. curl -X POST http://localhost:9090/-/reload

当你采用以上任一方式执行 reload 成功的时候，将在 promtheus log 中看到如下信息：

![reload_success.png](http://7o512j.com1.z0.glb.clouddn.com/reload_success.png)

如果有配置信息填写错误，将导致 reload 失败，你将看到类型如下信息：

```
ERRO[0161] Error reloading config: couldn't load configuration (-config.file=prometheus.yml): unknown fields in scrape_config: job_nae  source=main.go:146
```

提示： 我个人更倾向于采用 curl -X POST 的方式，因为每次 reload 过后， pid 会改变，使用 kill 方式需要找到当前进程号。


再分别说下这两种方式 Prometheus 内部实现：

第一种：通过 kill 命令的 HUP (hang up) 参数实现

首先 Prometheus 在 cmd/promethteus/main.go 中实现了对进程系统调用监听，
如果发现是 `syscall.SIGHUP` 的信号，那么就会执行 reloadConfig 函数。

代码类似:

```
hup := make(chan os.Signal)
signal.Notify(hup, syscall.SIGHUP)
go func() {
  for {
    select {
    case <-hup:
      if err := reloadConfig(cfg.configFile, reloadables...); err != nil {
        log.Errorf("Error reloading config: %s", err)
      }
    }
  }
}()
```

第二种：通过 web 模块的 `/-/reload` action 实现。

1. 首先 Prometheus 在 web(web/web.go) 模块中注册了一个 POST 的 action `/-/reload`,
它的 handler 是 `web.reload` 函数，该函数主要向 `web.reloadCh` chan 里面发送一个 `chan error`。
2. 在 Prometheus 的 cmd/promethteus/main.go 中有个 goroutine 来监听 web 的 reloadCh, 如果有值，那么执行 reloadConfig 函数.

代码类似：

```
hupReady := make(chan bool)
go func() {
	<-hupReady
	for {
		select {
		case rc := <-webHandler.Reload():
			if err := reloadConfig(cfg.configFile, reloadables...); err != nil {
				log.Errorf("Error reloading config: %s", err)
				rc <- err
			} else {
				rc <- nil
			}
		}
	}
}()
```

总结：Prometheus 内部提供了成熟的 hot reload 方案，这大大方便配置文件的修改和重新加载。
