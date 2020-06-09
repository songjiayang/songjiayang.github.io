---
title: "Translate English with Hacker Way"
date: 2016-03-04 22:31:00 +0800
tags: [效率]
---

背景：

ruby 开发对英文单词量比较高，像我这种半吊子英文水平，使用英汉字典翻译的时候较多。

前段时间看到 [@lingceng](https://ruby-china.org/lingceng) 写的 [bing_dictionary](https://github.com/lingceng/bing_dictionary) ，试用结果非常满意，速度快而且翻译比较准确。最近开始背诵单词，所以想将每天查询记录收集起来，以备每晚巩固之用, 故写了 [dict_cache](https://github.com/songjiayang/dict_cache)。

## 安装

```
gem install dict_cache
```

## 如何使用

#### 1. 翻译单词 （用到 bing_dictionary）

```
dict apple
```
![translate_word.png](http://7o512j.com1.z0.glb.clouddn.com/translate_word.png)

#### 2. 查询每日记录，默认是当天

```
dict_cache date:2016-03-04

dict_cache // today is default
```
![list_words.png](http://7o512j.com1.z0.glb.clouddn.com/daily_words)

#### 3. `dict_cache clear` 清空所有缓存记录

#### 4. `dict_cache --help/-h`  帮助

欢迎大家使用，有任何问题和想法，任何 [issue](https://github.com/songjiayang/dict_cache/issues) 和 pr 都 OK.
