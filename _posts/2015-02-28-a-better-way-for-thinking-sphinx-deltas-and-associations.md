---
layout: post
title: "A better way to reindex associations in thinking-sphinx"
category: technical
tags: [technical, thinking-sphinx, ruby, refactor]
imagefeature:
comments: true
share: true
---

众所周知，thinking-sphinx 提供了 Real-Time 和 Delta 两种索引，而我在项目中经常使用后者。
使用这种方式，通常你会遇到这样一个问题：

>> If you are using associations for field or attribute data, delta indexing will not automatically happen when you make changes to those association models. You will need to add a manual delta hook to make it all update accordingly.

举个例子：

Article model 有这样索引:

{% highlight ruby %}
indexes comments.content, :as => :comments
{% endhighlight %}

当 comments 有变化（更新或删除）， article.comments 是不会自动重新索引的; 为使其重新索引, 我们不得不 Do a hack：

{% highlight ruby %}
# comment.rb
after_save :set_article_delta_flag
after_destroy :set_article_delta_flag

# ...

private

def set_article_delta_flag
  article.delta = true
  article.save
end
{% endhighlight %}

代码很简单，满足需求，是否意味这个 fearch 就可以结束？ 当深入一想，这样的代码并不通用，因为一个项目会有很多类似模型，我们总不能将这样的代码到处 copy 吧。

<hr/>

我们的解决方案：

1.新建一个共用模块：

{% highlight ruby %}
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
{% endhighlight %}

2.在 ActiveRecord::Base 注入此模块
{% highlight ruby %}
# config/initializers/active_record_concerns.rb
ActiveRecord::Base.extend SphinxAssociationDeltaCallback

{% endhighlight %}


3.在需要的模型里通过 auto_touch_associations 方法，注入需要观察的关联对象：

{% highlight ruby %}
# comment.rb
auto_touch_associations :article
{% endhighlight %}

>> 总结：完成代码功能并非难事，但我们需要多一些思考，消除可见重复，让代码变的更好。
