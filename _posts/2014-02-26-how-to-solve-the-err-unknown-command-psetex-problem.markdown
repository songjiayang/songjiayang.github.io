---
title: How to solve the 'ERB unknown command psetex' problem
date: 2014-02-26 10:19:47 +0800
tags: [ruby]
---

After we upgraded sidekiq to the lastest version(v2.17.6) , we receive some error message.

```
A Redis::CommandError occurred in background at 2014-02-25 09:17:55 UTC :
ERR unknown command 'psetex'
.........
```

As you see the key word is **psetex**. What is *psetex*?

*psetex* is redis command, there is the [official introduction](http://redis.io/commands/*psetex*). It point out *psetex* is available since redis v2.6.0 。

I guess there is a redis version mistake in our server, i do a quickly check with command `redis-server -v`.  As expected the redis version is 2.2.12, it is too low. we must upgrade it .

How to upgrade redis to the lattest stable verison?

A example with v2.8.6

```
wget http://download.redis.io/releases/redis-2.8.6.tar.gz
tar vxzf redis-2.8.6.tar.gz
cd redis-2.8.6
make
sudo make install
```


After upgraded redis to a version above 2.6, the problem had been solved.

Happy, Cheers!
