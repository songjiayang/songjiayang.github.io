---
title: "为什么你的 Rails 不需要 JS 框架"
date: 2015-02-23 22:31:00 +0800
tags: [ruby]
---

现在前端的繁荣程度前所未有，尤其是前端 MVC 框架，多之又多，有点乱花渐入迷人眼的味道。

前端框架使用与否取决于你的应用形态，即 website 还是 web app。通常而言，网站为主的应用是不需要任何前端 mvc 框架的。
本文的意图不是为了排斥前端框架，而是为了讲述在没有使用任何前端框架的情况下，如何更好地组织项目的 js 代码。

#### 依赖的类库:
项目是按照 rails 4.2 官方推荐的方式，主要使用了 Turbolinks .

- jQuery.
- Coffeescript.
- Turbolinks.
- HeadJS, 使用 headjs-rails gem, 用来异步加载 js 代码，从而加快项目速度.

#### 代码组织结构:

![http://songjiayang.qiniudn.com/code_folder.png](http://songjiayang.qiniudn.com/code_folder.png)

- 一个页面对应了一个 js 视图类。
- 通用组件，全部放到 widget 目录里面。
- application.js 引入所有 js 依赖和项目自启动逻辑。

#### 视图代码

任何页面都继承于 ApplicationView ，如果没有属于自己的view,那么自动调用 ApplicationView.

```
window.Views ||= {}
class Views.ApplicationView
 render: ->
   Widgets.FancyBox.enable()
   Widgets.MarkdownEditor.enable()
 cleanup: ->
   Widgets.FancyBox.cleanup()
   Widgets.MarkdownEditor.cleanup()
```

因为我们要在很多个页面都使用 FancyBox 和 MarkdownEditor 这两个组建，所有直接就写在了ApplicationView里面。

特定的 view 直接继承 ApplicationView：

```
window.Views.Newsletters ||= {}
class Views.Newsletters.EditView extends Views.ApplicationView
 render: ->
   super()
   $('a.preview').click (e) ->
     e.preventDefault()
     url = $(e.target).attr('href')
     window.open(url, '_blank', 'width=800,height=800')
 cleanup: ->
   super()
```

够简单吧？ 为什么需要 clearup ?

因为我们使用了 `Turbolinks`，当有使用Turbolinks的时候，需要特别小心，因为当页面切换的时候 js 的环境不会被重置。
举个栗子： 如果在一个页面定义了一个 timer，切换到另外一个页面它也会被触发，所以，请记得在页面切换前，清空这个 timer, 而放在 clearup 只是为了统一管理。


#### 功能组件

这个没什么可讲的，和一般的视图类差不多。

```
window.Widgets ||= {}
class Widgets.FancyBox
  @enable:  -> $(".fancybox").fancybox()
  @cleanup: -> $(".fancybox").off()

```

#### 启动代码

application.js 只需要监听 Turbolinks 的事件，保证渲染当前页面对应的 view 并清空前一个页面的 view 即可。

```
#= require everything you need

pageLoad = ->
  className = $('body').attr('data-class-name')
  window.applicationView = try
    eval("new #{className}()")
  catch error
    new Views.ApplicationView()
  window.applicationView.render()

head ->
  $ ->
    pageLoad()
    $(document).on 'page:load', pageLoad
    $(document).on 'page:before-change', ->
      window.applicationView.cleanup()
      true
    $(document).on 'page:restore', ->
      window.applicationView.cleanup()
      pageLoad()
      true
```

显然这里需要一个特定的 data attribute 来表明当前页面对应的是哪个 view 。

我们在 `application_controller.rb` 中添加一个 `js_class_name` 方法:

```
 def js_class_name
   action = case action_name
     when ‘create’ then ‘New’
     when ‘update’ then ‘Edit’
    else action_name
   end.camelize
   "Views.#{self.class.name.gsub('::', '.').gsub(/Controller$/, '')}.#{action}View"
 end
```

在 layout 里面调用它：

```
%html
  %head
    = javascript_include_tag 'head.min'
    = headjs_include_tag 'application'
  %body{'data-class-name' => js_class_name}
```


#### 总结：

就像你所看到的，越来越多的 Rails 项目开始使用前端框架，例如 Angular, Ember 以及 Backbone。
诚然，它们确实是不错的选择，但是我希望你通过阅读此文后发现不使用任何前端框架也是一个不错的选择！

参考： [https://medium.com/@cblavier/rails-with-no-js-framework-26d2d1646cd](https://medium.com/@cblavier/rails-with-no-js-framework-26d2d1646cd)
