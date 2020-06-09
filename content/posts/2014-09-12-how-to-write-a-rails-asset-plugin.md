---
title: "How to Write a Simple Rails Assets Plugin"
date: 2014-09-12 22:31:00 +0800
tags: [ruby]
---

这里有一个 [如何编写Rails Plugins](http://guides.rubyonrails.org/plugins.html) 的官方教程。 但是由于 Assets 方面的gem都是简单的Css和Javascript内容，所以可以直接使用bundler 来创建。

前不久刚写了一个关于页面滚动至顶部的Gem, [scroll-up](http://rubygems.org/gems/scroll-up)。下面我会讲解这个Gem编写的大致步骤。

#### 第一步： 使用 bundler 生成Gem模版

```
bundle gem scroll-up
```}

#### 第二步：配置 .gemspec 文件

主要是添加这个gem的描述说明，以及作者信息。更多参数的意义可以参见 [http://guides.rubygems.org/specification-reference/](http://guides.rubygems.org/specification-reference/) 。

#### 第三步：继承 Rails::Engine

其实 Rails Plugins 都是继承于 ::Rails::Engine， 一般情况下 大家都会把这部分逻辑放在 engine.rb 这个文件中. 为了防止Gem之间的冲突，我们往往又会添加一个 Namespace, 例如：

```
# lib/scroll-up/engine.rb
module ScrollUp
  class Engine < ::Rails::Engine
  end
end
```

#### 第四步：Require Engine

因为Rails 的 bundle require 会自动加载 Gem 的 lib 目录下的所有 ＊.rb 文件，所以我们只要将 require 逻辑放在 lib 根目录任意一个 rb 文件中即可。例如：

```
# lib/scroll-up.rb
require 'scroll-up/engine'
```

#### 第五步：添加 Aseets 内容

Aseets 内容一般按照Rails默认的目录结构即可，scroll-up 是放在 app/assets 里面的：

```
// app/assets/javascripts/scroll-up.js

$(document).ready(function(){

  var scrollUp = function() {

    // Get link
    var link = $('#scrollUp');

    $(window).scroll(function() {
      // If the user scrolled a bit (150 pixels) show the link
      if ($(this).scrollTop() > 150) {
        link.fadeIn(100);
      } else {
        link.fadeOut(100);
      }
    });

    // On click get to top
    link.click(function() {
      $('html, body').animate({scrollTop: 0}, 200);
      return false;
    });
  };

  scrollUp();
}); // end document ready wrap

```

```
// app/assets/stylesheets/scroll-up.css
#scrollUp {
  position: fixed;
  display: none;
  background-color: #777;
  color: #eee;
  font-size: 40px;
  line-height: 1;
  text-align: center;
  text-decoration: none;
  bottom: 20px;
  right: 20px;
  overflow: hidden;
  width: 46px;
  height: 46px;
  border: none;
  opacity: .8;
}

#scrollUp:hover{
  color: #eee;
  background-color: #222;
}
```

#### 第六步：Build Gem

在 Gem 根目录执行:

```
gem build scroll-up.gemspec
```

#### 第七步：Publish Gem

在 Gem 根目录执行:

```
gem push scroll-up-xxx.gem
```
待 push 成功后，你就可以上 rubygems.org 这个网站去搜索自己的 Gem 了。


> 小提示：
使用 gem yank 命令可以删除已发布了的特定 gem 版本。例如：

```
gem yank scroll-up -v 0.0.3
```
