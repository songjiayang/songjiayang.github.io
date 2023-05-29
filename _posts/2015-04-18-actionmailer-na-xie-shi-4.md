---
title: "Actionmailer 那些事（四）"
date: 2015-04-18 23:31:00 +0800
tags: [ruby]
---

众所周知，通常情况 css 有三种插入方式，外链，内部和行内，详情请参考 [css_howto](http://www.w3schools.com/css/css_howto.asp),
但在 email 中只能使用行内（Inline Styles）的方式。 所以在日常开发中，我们的很多代码会写成这样：

![email-with-style-1.png](/images/email-with-style-1.png)

So ugly， So tired !

难道就没有什么方式，能让我们像写外链样式那样轻松写邮件模版吗？

当然有，这里我们需要先了解两个 gem ， [premailer](https://github.com/premailer/premailer) 和 [premailer-rails](https://github.com/fphilipe/premailer-rails)。

premailer 主要实现将页面定义的 css 样式自动转化为 Inline 的方式，官方说法是 *Each of these CSS declarations will be copied to appropriate element's attribute.*

测试代码：

```

require 'premailer'

premailer = Premailer.new('http://leemunroe.github.io/responsive-html-email-template/email.html', :warn_level => Premailer::Warnings::SAFE)

# Write the HTML output
File.open("output.html", "w") do |fout|
  fout.puts premailer.to_inline_css
end

# Write the plain-text output
File.open("output.txt", "w") do |fout|
  fout.puts premailer.to_plain_text
end

```

测试结果：

![email-with-style-2.png](/images/email-with-style-2.png)

通过测试结果可以看出，使用 premailer 可以轻松将定义的 css 样式拷贝到 dom 元素上。

premailer-rails 是 premailer 的 rails 插件，已集成到了 ActionMailer, 非常方便使用。

#### 1. 安装 gem

```
gem 'nokogiri'
gem 'premailer-rails'
```

#### 2. 邮件模版中引入 css 文件

```
<%= stylesheet_link_tag 'email'%>
```

开发模式下 `config.action_controller.asset_host` 一般会设置成 `http://localhost:3000`.
由于 premailer-rails 的一个小 bug, 具体请参见 [issues-138](https://github.com/fphilipe/premailer-rails/issues/138)。
所以这里需要一段 hack 代码：

```
<%if Rails.env.development?%>
<link href="/assets/email.css" media="screen" rel="stylesheet"/>
<%else%>
<%= stylesheet_link_tag :email%>
<%end%>
```

#### 3. preview 中查看结果

![email-with-style-3.png](/images/email-with-style-3.png)

Bingo,所有样式自动添加到了邮件 Dom 上，以后再也不用在邮件里面直接写 css 了。
更多用法请参考 [usage](https://github.com/fphilipe/premailer-rails#usage)。

代码示例: [https://github.com/songjiayang/rails-example/pull/2](https://github.com/songjiayang/rails-example/pull/2)
