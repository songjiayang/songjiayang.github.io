---
title: "Redmine lazy package"
date: 2015-02-26 22:31:00 +0800
tags: [效率]
---

####Why this repository
所有的一切都源自一篇博客 [5-redmine-plugins-will-change-way-you-work](http://it-consultis.com/blog/5-redmine-plugins-will-change-way-you-work)。
当我一个一个寻找这些插件时，发现这是一件超级浪费时间的事，更何况 [redminecrm](http://www.redminecrm.com) （一个redmine资源站点 ）
还被墙了 (>﹏<) 不～  ；所以，我决定将我常用的一些插件和皮肤汇总在这里，方便有需求的朋友使用。

项目地址： [https://github.com/songjiayang/redmine-lazy-package](https://github.com/songjiayang/redmine-lazy-package) 。

插件包括:

* [redmine_checklists](http://www.redminecrm.com/projects/checklist/pages/1)
* [redmine_agile](http://www.redminecrm.com/projects/agile/pages/2)
* [redmine_ timelog_ timer](https://github.com/emovere/redmine_timelog_timer)
* [redmine_ github _hook](https://github.com/koppen/redmine_github_hook)

ps：所有插件均成功跑在 redmine 3.0版本之上

#### How to use
1.将plugins 和 public/thmemes 的文件拷贝到你的 redmine 项目对应目录中。  
2.依次执行命令
```
bundle install --without development test
bundle exec rake redmine:plugins NAME=redmine_checklists RAILS_ENV=production
bundle exec rake redmine:plugins NAME=redmine_agile RAILS_ENV=production
```
##### 3. 重启 reminde，进入 ～/admin/plugins 和 ~/settings?tab=display 页面设置插件和皮肤。
