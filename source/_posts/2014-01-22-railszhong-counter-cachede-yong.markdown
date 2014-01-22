---
layout: post
title: "Rails中counter_cache的用"
date: 2014-01-22 17:27:30 +0800
comments: true
categories: [rails, counter_cache]
---

假设我们现在已经有了post.rb和comment.rb两个model, 如下：

```ruby app/models/post.rb
class Post < ActiveRecord::Base
  has_many :comments,  dependent: :destroy
end
```

```ruby app/models/comment.rb
class Post < ActiveRecord::Base
  belongs_to :postz
end
```

现在我们有这样一个需求： 统计每个文章评论数,这里 我们采用rails的`counter_cache`来处理。
现在我们开始一步一步使用rails 的counter_cache




