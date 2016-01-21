---
layout: post
title: "How to fix 'Nokogiri Incompatible library version' error in lion"
category: technical
share: true
comments: true
tags:
 - gem
---

今天将本地开发环境切换到 ruby 2.1.2 后，重新启动Rails的时候，遇到如下错误提示：

{% highlight bash %}

Incompatible library version: nokogiri.bundle requires version 11.0.0 or later,
but libxml2.2.dylib provides version 10.0.0

{% endhighlight %}

如何修复：

{% highlight bash %}
gem uninstall nokogiri libxml-ruby

brew update
brew uninstall libxml2
brew install libxml2 --with-xml2-config
brew link --force libxml2

brew install libxslt
brew link --force libxslt

bundle config build.nokogiri -- --with-xml2-dir=/usr --with-xslt-dir=/opt/local --with-iconv-dir=/opt/local
bundle install

{% endhighlight %}

参考链接： https://gist.github.com/vparihar01/5856524
