---
title: A non-invasive way adding js logic codes to activeadmin
date: 2014-08-12 22:31:00 +0800
tags: [ruby]
---

我们公司系统的后台管理使用的是[activeadmin](http://www.activeadmin.info/)这个gem,关于它的基本使用可以参考这里[官方文档](http://www.activeadmin.info/documentation.html)。

有时侯我们需要在activeadmin中实现一些比较酷的交互功能，例如表单select 改变的事件处理：

![select 联动动图](http://i.picasion.com/pic78/e137e46d88e92bec17b9f643511d1412.gif)

假如要实现以上功能，而又不想通过直接修改model中的后端逻辑代码的方式，那我们该怎么做呢？

- 答案就是采用纯js实现的方式。


#### 1、新建 active_admin/fields/form.coffee 文件

1. fields 是因为这个代码是针对`field`模型.
2. form.coffee是因为此功能只用在form表单处。


#### 2、 修改admin/field.rb模型，给form添加一个特定的seletor

```
form :html => { :class => "field_form"} do |f|
  f.inputs do
    # 此处省略了部分代码
  end
  f.actions
end
```

通过 `:html => { :class => "field_form"}` 给表单绑定了`field_form` 的 class

#### 3、 修改active_admin/fields/form.coffee文件，添加 js 逻辑代码

```
# FieldFormHelper 类定义
class FieldFormHelper

  # 定义一组 selectors ui
  ui:
    option_list_li: '#field_option_list_input'
    data_type_li: '#field_data_type_input'
    field_data_type: "#field_data_type"

  # 构造函数
  constructor: (el) ->
    @$el = $(el)

    # 缓存住所有需要用到的jquery selector
    @$option_list_li = @$el.find(@ui.option_list_li)
    @$data_type_li = @$el.find(@ui.data_type_li)
    @$field_data_type = @$el.find(@ui.field_data_type)

    #Show or Hide the  option_list
    @hideOrShowOptionList()

    # select 改变事件绑定
    @bindEvents()

  bindEvents: ->
    @$field_data_type.bind 'change',  =>
      @hideOrShowOptionList()

  hideOrShowOptionList: ->
    # 根据select的数值去判断到底是显示还是隐藏
    if @$field_data_type.find("option:selected").text() == 'Select'
      @$option_list_li.show()
    else
      @$option_list_li.hide()

# 给每一个field_form 添加属于自己的helper对象
$ ->
  $('.field_form').each (_ , el) ->
    new FieldFormHelper(el)

```

#### 4、 在active_admin.coffee 中加载刚写好的active_admin/fields/form.coffee文件

```
#= require active_admin/base
#= require_tree ./active_admin/fields  #直接加载了整个fields文件夹

```

至此，通过js的方式，我们已经完成了上述功能的实现。

通过此方式，避免了直接修改后端代码。我认为这种方式处理简单，代码也更好维护，您不妨一试！
