---
title: 使用 Plupload 创建 TempImage
date: 2014-01-11 19:57:14 +0800
tags: [ruby] 
---

[Plupload](http://www.plupload.com/)是一款比较流行的文件上传插件，能够轻易实现多文本上传,拖拽上传等效果，而且兼容性比较好。

什么是TempImage?

通常我们会将Image作为一个多态的资源,然后有一些特定的图片继承于它, 比如 `MaterialImage` (belongs_to Material).

假如Material在创建的时候需要上传一张图片，通常的做法是图片和表单数据一起上传。这样的做法有一个问题就是图片表单的提交会因为图片上传而耗费大量时间。

有什么办法可以解决这个问题呢？那就是采用TempImage的方式。具体而言就是先使用Plupload插件，异步将图片上传到后台，然后在表单中绑定新创建的图片的id即可。

```
 initialize_uploader: (options) ->

    ## CSRF Tokens Into Hash
    # Initialize a params hash
    @params = {}
    # Load csrf tokens from page
    csrf_name = $("meta[name='csrf-param']")?.attr('content')
    csrf_value = $("meta[name='csrf-token']")?.attr('content')
    # Load csrf tokens into params hash
    @params[csrf_name] = csrf_value

    ## Default options for the uploader
    options = $.extend(options, {
      url: "/images.json?temp=true"
      multipart_params: @params
      label: ''
      prompt: ''
      chunk_size : '1mb'
      browse_button: 'browser'
      unique_names : true
      multi_selection : false  ＃single or multi select
      max_file_size: '10mb'
      filters: [{ title: __("Image files"), extensions: "gif,png,jpg,jpeg,tiff"}]
    })

    ## Initialize a new uploader
    @uploader = new plupload.Uploader(options)
    @uploader.init()

    # This is a simple uploader; bind file upload directly after choice
    @uploader.bind 'FilesAdded', (uploader, files) ->
      uploader.start()

    # Once a file is uploaded, add the tempimage id to input
    @uploader.bind 'FileUploaded', (uploader, file, response) =>
      data = JSON.parse(response.response)?.image
      @browser || = @$el.find("#browser")
      @browser.attr('src',data.file_url)
      @temp_image_input ||= @$el.find('#temp_image_id')
      @temp_image_input.val(data.id)
```
