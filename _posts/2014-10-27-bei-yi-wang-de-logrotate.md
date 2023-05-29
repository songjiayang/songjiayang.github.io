---
title: 被遗忘的 logrotate
date: 2014-10-27 22:31:00 +0800
tags: [linux]
---

随着时间的推移，Application的日志内容越来越多（对我而言，主要是nginx 和 unicorn），而日志分析第一步往往是日志分片， 因为你不可能一次去处理几十天的日志，内容太多。

有些人可能会自己写一些定时任务来处理日志分片，其实这样做既费事，又不能保证与其他任务进程完美结合，而Linux 上自带的 [logrotate](https://github.com/stevendanna/logrotate) 却是一个比较好解决方案， 而且 logrotate 底层是依赖了 crontab, 很容易自定义任务周期。

logrotate 的配置文件默认都放在了 /etc/logrotate.d 里面。

其实 nginx 本身就做好了分片，下面是一段nginx 默认的 logrotate 配置。

```
# 文件位置是 /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    weekly
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    prerotate
        if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
          run-parts /etc/logrotate.d/httpd-prerotate; \
        fi \
    endscript
    postrotate
        [ -s /run/nginx.pid ] && kill -USR1 `cat /run/nginx.pid`
    endscript
}
```

我使用的 unicorn 的配置：

```
# 文件位置是 /etc/logrotate.d/unicorn_procution

# Modify the following glob to match the logfiles your app writes to:
/var/www/rails_app/shared/log/app_production.log {
    weekly
    missingok
        # keep 12 rotates
    rotate 12
    compress # must use with delaycompress below
        # add timestamp to log file
    dateext

    # this is important if using "compress" since we need to call
    # the "lastaction" script below before compressing:
    delaycompress

    # note the lack of the evil "copytruncate" option in this
    # config.  Unicorn supports the USR1 signal and we send it
    # as our "lastaction" action:
    lastaction
        pid=/var/www/rails_app/current/tmp/pids/unicorn.pid
        sudo test -s $pid && sudo kill -USR1 "$(cat $pid)"
    endscript
}
```

OK，按照上面的配置，已经实现了 nginx 和unicorn 日志分片了。

更多 logrotate 参数 的意义可以参考 [http://linux.die.net/man/8/logrotate](http://linux.die.net/man/8/logrotate)
