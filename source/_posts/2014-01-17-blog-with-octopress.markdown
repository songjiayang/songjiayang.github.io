---
layout: post
title: "Blog With Octopress"
date: 2014-01-17 13:48:15 +0800
comments: true
categories: [Blog, Octopress]
---


1. 出现类似 *YAML Exception reading index.markdown: invalid byte sequence in US-ASCII* 的错误时，只需要在执行命令前加上`LC_ALL="en_US.UTF-8"` 即可,如：
{%codeblock lang:ruby %}
LC_ALL="en_US.UTF-8" rake generate
{%endcodeblock%}

