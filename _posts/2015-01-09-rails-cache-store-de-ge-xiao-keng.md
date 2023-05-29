---
title: "rails cache store 的一个小坑"
date: 2015-01-09 22:31:00 +0800
tags: [ruby]
---

曾然，标题有点哗众取宠了，其实这并不是个 Rails 的坑，只是一个我平时没注意到的问题罢了。

想必大家对 Rails 的 `ActiveSupport::Cache::Store ` 已经很了解了，是吗？那你觉得下面这段代码有什么问题。

```
# @user is a ActiveRecord::Base instance
Rails.cache.fetch([@user]) do
  # data decorate
end
```

如果你已经找出问题了，那恭喜你，可以跳过下面的内容了。

<hr/>

上面代码的问题就是：fetch 的参数应该是 `@user` 非 `[@user]`， 为什么？

尽管这里使用 `[@user］` 不会抛出异常，但是它并不会在 @user 变化后，自动更新上面代码中的缓存数据，这显然与我们想要的自动过期策略不符。

为什么将 `[@user]` 换成 `@user` 就可以？

那就要分析下 Rails 的源代码了.

首先看 Rails 的 fetch 方法的定义：

```
def fetch(name, options = nil)
  if block_given?
    options = merged_options(options)
    key = namespaced_key(name, options)

    cached_entry = find_cached_entry(key, name, options) unless options[:force]
    entry = handle_expired_entry(cached_entry, key, options)

    if entry
      get_entry_value(entry, name, options)
    else
      save_block_result_to_cache(name, options) { |_name| yield _name }
    end
  else
    read(name, options)
  end
end
```
其中与 key 相关的是 namespaced_key 这个方法：

```

# File activesupport/lib/active_support/cache.rb, line 532
def namespaced_key(key, options)
  key = expanded_key(key)
  namespace = options[:namespace] if options
  prefix = namespace.is_a?(Proc) ? namespace.call : namespace
  key = "#{prefix}:#{key}" if prefix
  key
end
```

`namespaced_key` 里面调用了 `expanded_key` 方法：

```
# File activesupport/lib/active_support/cache.rb, line 513
def expanded_key(key) # :nodoc:
  return key.cache_key.to_s if key.respond_to?(:cache_key)

  case key
  when Array
    if key.size > 1
      key = key.collect{|element| expanded_key(element)}
    else
      key = key.first
    end
  when Hash
    key = key.sort_by { |k,_| k.to_s }.collect{|k,v| "#{k}=#{v}"}
  end

  key.to_param
end
```

OK，到这里，你该知道问题了，`[@user]` 和 `@user` 分别生成了怎样的 key，你也一定了解了吧。

出现这个问题核心是，如果生成的key的相关因子只有1个，写成 [@user] 这样的形式，
key的生成，最终调用的其实只是是 `to_param` 这个方法，而它始终 == `"#{@user.id}"`，所以key一直不会有变化。

<hr/>

结语：当 cache key 的因子只有1个时，不要使用数组。
