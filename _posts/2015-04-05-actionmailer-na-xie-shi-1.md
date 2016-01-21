---
layout: post
title: "ActionMailer 那些事(一)"
category: technical
comments: true
share: true
description: "from address with aliases"
tags:
 - ror
 - ActionMailer
---

上周遇到这样一个需求，系统邮件发送至客户，客户看到的邮件 from 地址不应该是直接的 email,而应该是email对应的一个别名。如图：

<img src="{{ site.url }}/images/mailer-from.png">

在 Rails 如何实现？其实只需将邮件发送的 from 参数 写成 `"NAME <EMAIL>"` 的形式即可：

{% highlight ruby%}

class NotifyMailer < ActionMailer::Base

  def new_user()
    xxxxx
    mail(from: "Matter.build <no-reply@matter.build>", .....)
  end

end
{% endhighlight%}

如果你想统一配置 from 的格式，只需添加 `ActionMailer::Base.default` 默认值即可，例如：
{% highlight ruby%}
#config/application.rb
...
ActionMailer::Base.default :from => 'Matter.build <no-reply@matter.build>'
...
{% endhighlight %}

推荐阅读： [ActionMailer 那些事(二)](/technical/actionmailer-na-xie-shi-2)
