---
layout: post
title: "A better way to reindex thinking-sphinx's associations"
category: archive
description: ""
tags:
 - thinking-sphinx
---

众所周知，thinking-sphinx 提供了 Real-time Indexes 和 Delta Indexes 两种索引方式，而我在项目中经常使用的是后者。
使用这种方式，通常你会遇到这样一个问题：

>> If you are using associations for field or attribute data, delta indexing will not automatically happen when you make changes to those association models. You will need to add a manual delta hook to make it all update accordingly.

举个栗子：

Article model 中有这样一个索引:

```
indexes comments.content, :as => :comments
```

当 article 的 comments 改变（更新或删除）的时候， article 是不会自动重新索引所有评论内容的，为了达到自动索引的效果，我们不得不在 comment 里面做一些额外操作：

``` ruby
# comment.rb
after_save :set_article_delta_flag
after_destroy :set_article_delta_flag

# ...

private

def set_article_delta_flag
  article.delta = true
  article.save
end
```

代码逻辑很简单，也满足了需求。但往往一个项目中很多模型都会有同样问题，那我们该怎么办？有没有比较好的方式，避免重复写相似的代码。

下面是我们的解决方案：

对应文件添加如下代码：

```ruby
# app/models/concerns/auto_touch_delta_flag.rb
module AutoTouchDeltaFlag

  def auto_touch_associations(*associations)
    set_delta_flag = proc do
      associations.each do |association|
        association_model = self.send(association)
        association_model.delta = true
        association_model.save
      end
    end

    after_save set_delta_flag
    after_destroy set_delta_flag
  end
end
```

```ruby
# config/initializers/active_record_concerns.rb
ActiveRecord::Base.extend SphinxAssociationDeltaCallback

```


有了上面的代码，comment 中只需添加一行代码就实现相同功能，而且这种方式可以很方便扩展到其他模型：

```ruby
# comment.rb
auto_touch_associations :article
```
