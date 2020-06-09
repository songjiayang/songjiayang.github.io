---
title: Rails中 counter_cache 的使用
date: 2014-01-22 17:27:30 +0800
tags: [ruby] 
---

假设我们现在已经有了post和comment两个model:

```
#app/models/post.rb
class Post < ActiveRecord::Base
  has_many :comments,  dependent: :destroy
end
```

```
#app/models/comment.rb
class Comment < ActiveRecord::Base
  belongs_to :post
end
```

有这样一个需求：统计每个文章评论数。这里我们采用rails的`counter_cache`来处理。

我们在以上基础一步一步地添加 rails 的 counter_cache 的使用。

1.在Post中添加一个**comment_count**的字段来存储comments的数量,可以直接使用migration来处理:

```
rails g migration add_comment_count_to_post comment_count:integer
```

2.修改comment.rb文件，添加counter_cache的支持:  

```
#app/models/comment.rb
class Comment < ActiveRecord::Base
  belongs_to :post, counter_cache: :comment_count
end
```

3.修改rspec测试用例:

```
#spec/models/post.rb
context "comment_count" do
  let(:post){create :post}
  let(:comment1){ create :comment, :post => post}

  it "comment_count should +1" do
    expect {
        create :comment, :post => post
    }.to change{post.reload.comment_count}.by(1)
  end

  it "comment_count should -1" do
    expect {
       comment1.destroy
    }.to change{post.reload.comment_count}.by(-1)
  end
end
```

4.使用rake来处理历史数据:

```
#lib/tasks/set_post_comment_count.rake
namespace :post do
  desc "set post's comment count"
  task :set_comment_count => :environment do
    Post.update_all :comment_count => 0 #先清零
    Post.find_each(batch_size: 5000) do |post|
      Post.update_counters post.id, :comment_count => post.comments.count
    end
  end
end
```

最后在在项目根目录执行`rake post:set_commnet_count` .
