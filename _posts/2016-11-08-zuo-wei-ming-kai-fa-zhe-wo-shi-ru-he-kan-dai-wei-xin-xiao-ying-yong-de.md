---
title: "作为一名开发者，我是如何看待微信小应用的"
date: 2016-11-08 23:31:00 +0800
tags: [微信]
---
![微信小应用](http://7o512j.com1.z0.glb.clouddn.com/Screen%20Shot%202016-11-07%20at%2010.18.17%20PM.png)

微信小应用终于开放内测了，一时间网络上炸开了锅，对此评论褒贬不一。

有的把它说的无比万能，是各种 App 的终结者, 也有把它看衰的非常彻底，觉得就是张小龙实验的玩具。

那么作为一名立志站在技术前沿的开发者，我们不能被网络上的各种嘴炮所忽悠，对微信小应用应保有自己的判断和认识。

以下是我想与大家分享的一些看法：

好的方面：

上手容易，微信小应用使用了目前非常火的 js  native 的方案，意味着无论你是从 react+native 还是 weex  过来，都不会太大的技术负担。其实微信小应用就是在这些社区方案中，摸索并标准化了一套自己的方案而已,开发技术和常见的 Web 并没太多差异。

文档清晰，我想说这次小应用的文档绝非外包，我花费了并不多的时间，就把所有想要了解的东西都了解了完全，而且基本都找到了示例代码，这个和微信公众号文档比起来，简直爽的不要不要的。

API 丰富，有人说，微信小应用，只能做做简单的表单应用，这一点，我不认同。对于常规的应用，可能一般的 UI 组件就能满足，比如图片，列表，表单等；但对于一些复杂功能，微信小应用的 API 同样可以满足。你不仅可以使用 websocket （同时只能存在一个 websocket）实现实时消息，使用 cavans 绘图，还能使用地图，重力感应接口来做更多有趣的事情。

UI 统一，微信提供了统一的 UI 接口，意味着你只需要写一套代码就可以运行在不同的平台了，平台的差异性，让微信去解决吧。微信小应用中引入了 flex 和 rpx (responsive pixel)，意味着在处理响应式布局方面，变得更加容易。

作为开发者，我们终于可以从各平台差异性以及不同分辨率适配中释放出来，更多的关注功能和引流。

应用市场，相较于同时上架苹果 App 商城，以及国内各大主流 Android App 商城，现在你只需要在微信APP 商城上架一次，这显得更加方便而敏捷。

对于用户而言，管理自己的应用也变得方更加简单，不仅因为 App 下载更容易，花费时间更少，而且还因为可以将用户 App 和微信帐号绑定，实现跨手机的云端同步。

不好的方面：

接口开放力度不够，或许是因为微信对小应用的一些额外担忧，像一些对于引流非常关键功能，比如朋友圈分享，微信并没有开放。这意味着用户只能通过扫描二维码或则使用微信内置的 App 搜索了解和下载应用。这样对于开发者而言，引流和变现是个比较大的问题。

因为微信小应用才刚开始，待其运营成熟以后，微信会不会慢慢开放接口，我们拭目以待。

完全放弃支付宝支付功能，大家都知道支付宝和微信不能共存，选择了微信小应用，意味着你就自动放弃了支付宝，这个对于一些电商应用，是个比较大的打击。

以上，就是我对于微信的一些个人看法，作为一名开发者，我觉得微信小应用具有还是具有很高的创新价值，此时我突然想到了 JVM，不知不觉陷入了一种 write once, run anywhere 的 yy 中。

至于微信小应用能不能火起来，我不敢说，这取决于机遇和微信团队的运营能力，但是对于新的事物，我们应该多一点包容，多一点尝试，个人持乐观态度。

最后，祝大家工作更顺利，生活更美好。

是的，明天又是美好的一天！