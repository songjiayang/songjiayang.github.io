---
title: "Thinking-sphinx can not search chinese"
date: 2014-11-20 23:31:00 +0800
tags: [ruby]
---

今天发现 [Matter](www.matter.build) 在生产环境不支持中文全文搜索（英文可以），开发环境支持中文搜索。

Matter 的全文搜索使用的是 [thinking-sphinx](https://github.com/pat/thinking-sphinx)，
底层是 sphinxsearch 。

由于习惯性思维，我很快的查看了线上的 sphinxsearch 版本，
发现版本比较低， 所以立马升级了线上的 sphinx 版本(与开发一致)。
很不幸，不是这个原因，问题犹在，需要调整思路。

经过对比线上和本地自动生成的 x.sphinx.conf 文件，发现本地
生成的配置文件多了 `sql_query_pre = SET NAMES utf8` 这句。

也许问题就出现在这里，那为什么开发环境会多处这句配置？

通过在 github 的 thinking-sphinx 项目里面搜索 [SET NAMES utf8](https://github.com/pat/thinking-sphinx/search?utf8=%E2%9C%93&q=utf8_query_pre&type=Code)
和 [utf8_query_pre](https://github.com/pat/thinking-sphinx/search?utf8=%E2%9C%93&q=utf8_query_pre&type=Code)
瞬间恍然大悟！

原来线上的 database.yml 里面并没有设置编码方式，而默认编码并非utf8，所以不支持中文。
而开发环境支持中文搜索的原因正是在 database.yml 里面设置了 `encoding: utf8`。
待生产环境加上这句后，问题得以解决，可以正常搜索中文了。

这个bug的修复并非如此顺利，因为一开始浪费了很多时间在其他方向上，最后在同事
[serco-chen](https://github.com/serco-chen) 的提醒下，通过对照配置文件差异后，才得以解决，在此感谢他。

通过此次捉虫，感触最深的就是：
>>当我们调试和解决bug的时候，不妨尝试采用对照的方式，切勿先入为主，以为是必然就忽略了某些细节。
