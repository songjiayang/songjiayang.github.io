---
title: "Rail日历小插件"
date: 2015-08-17 23:31:00 +0800
tags: [ruby]
---

昨天参加完上海 RailsGril,遇到一个日历打印的问题，及输入任何一个日期，打印出当月的日历。

写了一段 Rails 小代码:
```
class  Calendar
   attr_reader :time

   def initialize(time=Time.now)
      @time = time
      unless time.is_a? Time
        @time = Time.parse time
      end
   end

   def render
    all_days = render_days
    puts '日|一|二|三|四|五|六'
    all_days.each_slice(7).to_a.each do |days_in_week|
      days_in_week.map!{|day| day.length == 1? "#{day} " : day}
      puts days_in_week.join('|')
    end
    nil
   end

   def render_days
     Array.new(time.at_beginning_of_month.wday, '  ') + (1..Time.days_in_month(time.month)).map(&:to_s)
   end
end
```

执行代码输出结果如下：

```

Calendar.new.render

日|一|二|三|四|五|六
  |  |  |  |  |  |1
2 |3 |4 |5 |6 |7 |8
9 |10|11|12|13|14|15
16|17|18|19|20|21|22
23|24|25|26|27|28|29
30|31

calendar = Calendar.new('2015/07/01')
calendar.render

日|一|二|三|四|五|六
  |  |  |1 |2 |3 |4
5 |6 |7 |8 |9 |10|11
12|13|14|15|16|17|18
19|20|21|22|23|24|25
26|27|28|29|30|31

```

总结： 有几个关于 Time 的 helper 方法， 比如 time.at_beginning_of_month， Time.days_in_month 的使用。
