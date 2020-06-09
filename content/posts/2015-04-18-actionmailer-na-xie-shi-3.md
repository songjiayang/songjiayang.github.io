---
title: "Actionmailer 那些事（三）"
date: 2015-04-18 22:31:00 +0800
tags: [ruby]
---

早已知道 Rails4 的 ActionMailer 有一个 email preview 的功能，但由于各种原因从未真正使用，其实它是的好东西，绝非鸡肋。

我们都知道项目中邮件发送是一件非常耗时且依赖配置的过程。在开发阶段，邮件模版需不断修改，
如果每次修改都要真正发送邮件才能确定效果，这真是一件糟糕的事情。因为你花在等待，刷新邮件客户端，排查收不到邮件原因（测试账号服务商邮件发送莫名延迟或今日邮件发送次数限制）的时间将足够你喝杯咖啡了。

Rails4 推出的 email preview 就是为了解决这个问题，使用它，你可以在浏览器中通过 url 访问， 直接获取邮件内容（和邮件客户端收到效果一模一样）。

时间就是生命，既然有这么好的东西，为何不用？下面小鱼将带着大家一步一步使用它。


### ActionMailer Preview 简单使用

1.使用命令 `rails g mailer UserMailer new_user` 创建 一个 mailer 和 action

2.此时，系统将自动未我们创建对应的 preview controller，打开`.../user_mailer_preview.rb`文件，你会发现 rails 给出的提示是如此清晰：
```

# Preview all emails at http://localhost:3000/rails/mailers/user_mailer
class UserMailerPreview < ActionMailer::Preview

  # Preview this email at http://localhost:3000/rails/mailers/user_mailer/new_user
  def new_user
    UserMailer.new_user
  end

end

```

3.启动 server, 在浏览器中输入 `http://localhost:3000/rails/mailers/user_mailer/new_user` 即可看到对应邮件内容。

![email_preview](/images/email_preview.png)

备注： 你可以按照具体业务逻辑任意修改 UserMailerPreview 中的 new_user 方法,例如一些参数装配，只要返回结果是一个 ActionMailer::MessageDelivery 即可。

其实，在日常邮件开发过程中，除了使用 preview 加快开发速度，还应配合测试先行的策略，大量排错，减少必须肉眼发现错误的可能性。
而常规的邮件测试至少应关心 邮件的 title, to, from 是否正确, body内容是否正确渲染（如果没有异常），模版大概是这样：

```

require 'test_helper'

+class UserMailerTest < ActionMailer::TestCase
  test "new_user" do
    mail = UserMailer.new_user
    assert_equal "New user", mail.subject
    assert_equal ["to@example.org"], mail.to
    assert_equal ["from@example.com"], mail.from
    assert_match "Welcome to my website!", mail.body.encoded
  end
end

```

需运行的测试命令是 `rake test` or `rake test %{FILE_NAME}` 。


参考: [http://api.rubyonrails.org/v4.2.1/classes/ActionMailer/Base.html#class-ActionMailer::Base-label-Previewing+emails](http://api.rubyonrails.org/v4.2.1/classes/ActionMailer/Base.html#class-ActionMailer::Base-label-Previewing+emails)

温馨提示： 我在github上新开了一个 rails-example 项目，用于平时博客代码演示，例如这篇博客代码，在[这里](https://github.com/songjiayang/rails-example/pull/1/files) 。
