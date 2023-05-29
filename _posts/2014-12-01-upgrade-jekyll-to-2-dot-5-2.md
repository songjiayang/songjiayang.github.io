---
title: "Upgrade jekyll to 2.5.2"
date: 2014-12-01 23:31:00 +0800
tags: [效率]
---


今天将自己博客生成器 [jekyll](!https://github.com/jekyll/jekyll) 由 2.4.0 升级到了 2.5.2。

升级代码：

```
gem update jekyll -v 2.5.2
```

升级完毕后，在构建博客的时候，出现 ` cannot load such file -- mercenary/program (LoadError) ` 的错误。

解决办法：使用 bundle exec 运行 jekyll ，或者切换到项目根目录，重新执行 bundle install。

参考资料： https://github.com/mmistakes/skinny-bones-jekyll/issues/10
