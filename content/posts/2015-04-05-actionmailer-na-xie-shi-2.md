---
title: "ActionMailer 那些事 (二)"
date: 2015-04-05 23:31:00 +0800
tags: [ruby]
---

这篇博客是上一篇 [ActionMailer 那些事(一)](/technical/actionmailer-na-xie-shi) 的续。

如你所知，上一篇博客提到邮件发送的 from 别名设置，其实，正常情况下，邮件from地址可以任意填写，
*一般邮件服务器*是不检查 from 值的（只要包含email格式）。

**为什么会有任意填写 from 地址的需求？**  

因为有时系统会代替客户发送邮件，而期望邮件接受者可以直接通过邮件回复发送的用户，此时，邮件的 from 就该填写用户的 email 了。

本来这并不是问题，因为你只要将 from 值修改成用户的 email 皆可，对于绝大多数邮件服务商这样是可以的，`但是 gmail 不行`。

**为什么不行？**

首先 gmail 处于自己的考虑，必须保证 from 的地址是配置发送邮件账号的 email 地址;
它可以通过简单的正则替换就搞定这个事情，毕竟所有邮件它都有能力修改。

**解决办法？**

不要使用 gmail 作为应用邮件发送服务器，毕竟它本来就不干这个事情，你应该选择那些邮件专业服务器提供厂商，
例如 SendGrid, Mailgun 等。

> 友情提示一下： heroku上面邮件服务器，如果配置为 gamil，将无法发送邮件，因为 gmail 已经封了所有来自 heroku 的认证请求。

推荐阅读： [ActionMailer 那些事(一)](/technical/actionmailer-na-xie-shi)
