---
layout: post
title: "redmine lazy package"
category: archive
description: "It is a redmine plugin and theme collection, all of them are very useful."
tags:
 -
---

我在 github 上新建了一个项目 [redmine-lazy-package](https://github.com/songjiayang/redmine-lazy-package),它主要用于搜集我常用的redmine 插件和皮肤。

#### Why this repository
所有的一切都源自一篇博客 [5-redmine-plugins-will-change-way-you-work](http://it-consultis.com/blog/5-redmine-plugins-will-change-way-you-work)。
当我一个一个寻找它们的时候，才发现这时一件蛮费时间事情，更何况 [redminecrm](http://www.redminecrm.com) （一个最大redmine资源的站点 ） 还被墙了；所以，我决定将我常用的一些插件和皮肤汇总在这里，方便有需求的朋友使用。

插件包括:

* [redmine_checklists](http://www.redminecrm.com/projects/checklist/pages/1)
* [redmine_agile](http://www.redminecrm.com/projects/agile/pages/2)
* [redmine_ timelog_ timer](https://github.com/emovere/redmine_timelog_timer)
* [redmine_ github _hook](https://github.com/koppen/redmine_github_hook)

ps：所有插件均成功跑在 redmine 3.0版本之上

#### How to use
#####1. plugins 和 public/thmemes 的文件拷贝到你的 redmine 项目对应目录中。
#####2. 依次执行命令
```ruby
bundle install --without development test
bundle exec rake redmine:plugins NAME=redmine_checklists RAILS_ENV=production
bundle exec rake redmine:plugins NAME=redmine_agile RAILS_ENV=production
```
##### 3. 重启 reminde，进入 ～/admin/plugins 和 ~/settings?tab=display 页面设置插件和皮肤。
