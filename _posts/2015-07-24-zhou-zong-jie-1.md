---
title: "周总结 《一》 "
date: 2015-07-24 22:31:00 +0800
tags: [ruby]
---

### 一. 使用 `rm` 命令删除大文件问题

背景： 测试机硬盘空间不足，需要删除部分大文件。使用 rm 命令 **成功** 删除这些文件后，
再通过 `df` 查看，磁盘空间依然不足。

原因： 这些删除的文件，被某些进程标记使用着，所以即使文件删除，这段释放的硬盘依然被标记不可用，磁盘空间并未真正释放。

解决办法： 1. 使用 ` echo "" > ` 命令取代 rm 。 2. 删除文件后，kill 掉对应进程 。  

### 二. thinking-sphinx 关联表索引建立问题

比如有如下索引：

```
#project_material_index.rb
ThinkingSphinx::Index.define :project_material, :with => :active_record, :delta => ThinkingSphinx::Deltas::SidekiqDelta do
.....
indexes :name, :sortable => true
indexes "name IS NULL", :as => :name_nil, :sortable => true
indexes tags.name, :as => :tag_names, :sortable => true
.....
end
```

重建索引，得到 ` Column 'name' in field list is ambiguous` 错误信息。

原因： 多张表关联索引，出现同名字段，sphinx 在生成索引 sql 语句时，不知道该使用哪个表中的字段。

解决办法： 显示添加表名限定即可，

修改后的代码如下：

```
ThinkingSphinx::Index.define :project_material, :with => :active_record, :delta => ThinkingSphinx::Deltas::SidekiqDelta do
.....
indexes :name, :sortable => true
indexes "project_materials.name IS NULL", :as => :name_nil, :sortable => true
indexes tags.name, :as => :tag_names, :sortable => true
.....
end
```

请注意代码中 `project_materials.name IS NULL` 的变化。

### 三. godaddy ssl 证书续费问题

续费 godaddy ssl 证书，到网站上下载最新的 key ，压缩后得到两个文件:

![godaddy](/images/godaddy.png)

使用： 将较小文件的内容完整 copy 到大文件的头部，然后用合并过后的文件替换服务器上的 xx.bundle.crt 文件即可。
