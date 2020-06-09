---
title: "说说那些网站 console 里的『 彩蛋 』"
date: 2016-01-21 23:31:00 +0800
tags: [有趣]
---

下图分别是淘宝，百度，京东网站 console.log 输出的内容，（ 腾讯网站无 ）。

![taobao](/images/caidan/taobao.png)
![baidu](/images/caidan/baidu.png)
![jd](/images/caidan/jd.png)
今天我可不是来给他们打广告的。

咳咳 ......

其实，你注意到没有，他们的打印输出内容是有样式的，更有甚至打印图片。
![caidan-image](https://camo.githubusercontent.com/0f25ac62249194f32edbd54c912ae2952aa02f1a/687474703a2f2f692e696d6775722e636f6d2f4f646f564d44532e706e67)

好了，我们用代码如何实现类似效果 ？  

答案就是使用 console.log 的 `%c` 参数（ 及 css style 打印输出 ）。

打印上图淘宝内容对应代码大致为：

![code.png](/images/caidan/code.png)
后记： 原来大公司也喜欢在自己网站留下一些好玩的 『 彩蛋 』 ，不过好奇，因为这个京东一年会收到多少封简历。

参考链接：

-  [https://developer.chrome.com/devtools/docs/console-api#consolelogobject-object](https://developer.chrome.com/devtools/docs/console-api#consolelogobject-object)
-  [http://getfirebug.com/wiki/index.php/Console.log](http://getfirebug.com/wiki/index.php/Console.log)
- [https://github.com/adriancooney/console.image](https://github.com/adriancooney/console.image)
