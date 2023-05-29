---
title: "mysql performance 之 filesort"
date: 2014-12-19 22:31:00 +0800
tags: [mysql]
---

昨天遇到一个 Mysql 大表(单表700万）查询的问题。这个问题是在开发环境中 Rails 的终端打印日志中发现的，内容大概是这样：

![mysql-filesort-1.png](http://7o512j.com1.z0.glb.clouddn.com/mysql-filesort-1.png)

根据日志信息，你不难发现，这是个 mysql `filesort` 的问题，如果你还不太了解，可以参考这篇 [博客](http://blog.csdn.net/yangyu112654374/article/details/4251624)。

既然是索引不当，那么就要修改，开始排查 readings  的所有索引。

很快我就定位到了一个叫做 `index_readings_on_reading_time_and_reporting_device_id` 的索引，
它是通过 rails 的 migrate 加上的：

```
add_index :readings, [:reading_time, :reporting_device_id]
```

实际上，在 readings 上最终执行的查询语句类似这样：

```
SELECT * FROM readings WHERE reporting_device_id = ?
AND (reading_time BETWEEN ? AND ?) ORDER BY reading_time DESC
```

很明显，上述查询先执行 WHERE 再执行 ORDER BY， 可是，我们的索引建立顺序，刚好相反。

那如何修复，很简单，动手写一个 migration：

```
class ChangeIndexOnReadings < ActiveRecord::Migration
  def up
    remove_index :readings, [:reading_time, :reporting_device_id]
    add_index :readings, [:reporting_device_id, :reading_time]
  end

  def down
    remove_index :readings, [:reporting_device_id, :reading_time]
    add_index :readings, [:reading_time, :reporting_device_id]
  end
end

```

这样修改真的管用吗？db:migrate 后，再次测试，结果如下：

![mysql-filesort-2.png](http://7o512j.com1.z0.glb.clouddn.com/mysql-filesort-2.png)

其实结果还是很明显的，filesort 没有了，速度也加快了不少。

OK，问题解决了，那有什么值得总结的呢？我认为有以下2点：

1. Mysql filesort 优化技巧。
2. Rails 在创建表的过程中，不要随意创建索引。索引的创建都应该与具体业务（查询）相关，尤其是那些组合索引，有时索引位置不当会事得其反。通常有且只有表的外健是在表创建的时候直接加上索引的。
