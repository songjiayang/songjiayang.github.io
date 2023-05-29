---
title: 'The Problem "Uncaught Error: Error calling method on NPObject" with plupload-rails'
date: 2014-11-12 22:31:00 +0800
tags: [ruby]
---

昨天，突然发现在 IE 上传图片的时候，报出错误 `Uncaught Error: Error calling method on NPObject`,
google了一下，发现这个是 flash 抛出来的错误，主要是跨站的问题。

####为什么会跨站呢？
首先我们使用qiniu作为静态资源的镜像加速，所以用于上传的插件也来自七牛。
主要涉及的资源是 Moxie.swf（flash 使用） 和 Moxie.xap（silverlight） 这两个。

plupload-rails 中关于资源加载代码是这样的：

```
// Assigns default Plupload settings that work with the asset pipeline.
(function ($) {
  var proxied = plupload.Uploader;

  plupload.Uploader = function (settings) {
    settings = $.extend({}, settings, {
      multipart: true,
      flash_swf_url: '<%= asset_path("Moxie.swf") %>',
      silverlight_xap_url: '<%= asset_path("Moxie.xap") %>'
    });

    return proxied.call(this, settings);
  };

  plupload.Uploader.prototype = proxied.prototype;
}(jQuery));

```

很显然，问题出现在 `asset_path("Moxie.swf")` 这句，
其实，Moxie.swf 和 Moxie.xap 并不需要镜像加速,
所以我们将这两个文件放在 项目public，直接使用即可。

所有直接修改上面代码即可：

```
flash_swf_url: '/misc/Moxie.swf',
silverlight_xap_url: '/misc/Moxie.xap'
```

备注：Moxie.swf 和 Moxie.xap 放在了项目下的 public/misc 文件夹中。

最后在 application.js 中将 plupload.settings 替换成修改后的文件即可.
我习惯将修改后的文件加个 override， 所以需要将
`//= require plupload.settings` 替换成 `//= require plupload.settings.override`.

最后 plupload.settings.override 代码是这样：

```
// Assigns default Plupload settings that work with the asset pipeline.
(function ($) {
  var proxied = plupload.Uploader;

  plupload.Uploader = function (settings) {
    settings = $.extend({}, settings, {
      multipart: true,
      flash_swf_url: '/misc/Moxie.swf',
      silverlight_xap_url: '/misc/Moxie.xap'
    });

    return proxied.call(this, settings);
  };

  plupload.Uploader.prototype = proxied.prototype;
}(jQuery));

```
