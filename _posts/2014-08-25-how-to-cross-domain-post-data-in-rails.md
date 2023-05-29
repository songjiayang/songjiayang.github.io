---
title: How To Cross-Domain Post Data In Rails
date: 2014-08-25 22:31:00 +0800
tags: [ruby]
---

今天帮同事做了一个邮件订阅的功能，订阅功能本身并不复杂；但是由于订阅页面是放在GitHub Page上的，所以涉及到浏览器中跨域提交数据的问题。

这里我们大致可以将服务需求分为客户端和服务端。 客户端是一个Github静态页面，你可以通过 [votenotes.com](http://votenotes.com/) 访问；服务端是一个提供API调用的Rails。

#### 1、客户端代码, 设置Ajax请求参数crossDomain为true
```
$.ajax({
  type: 'POST',
  crossDomain: true, // sert crossDomain true
  url: 'http://localhost:3000/sub_emails',
  data: data ,
  dataType: 'json',
  success: function(responseData) {
    //省略
  }
});
```

#### 2、修改Rails API代码，去除token验证并设置 CORS ACCESS HEADER

```
class SubEmailsController < ApplicationController

  skip_before_filter :verify_authenticity_token

  before_filter :cors_preflight_check
  after_filter :cors_set_access_control_headers

  def create
    SubEmail.create(email: params[:email])
    render status: 200, json: { message: "Sub Successful" }
  end

  private

  def cors_set_access_control_headers
    headers['Access-Control-Allow-Origin'] = '*'
    headers['Access-Control-Allow-Methods'] = 'POST, GET, PUT, DELETE, OPTIONS'
    headers['Access-Control-Allow-Headers'] = 'Origin, Content-Type, Accept, Authorization, Token'
    headers['Access-Control-Max-Age'] = "1728000"
  end

  def cors_preflight_check
    if request.method == 'OPTIONS'
      headers['Access-Control-Allow-Origin'] = '*'
      headers['Access-Control-Allow-Methods'] = 'POST, GET, PUT, DELETE, OPTIONS'
      headers['Access-Control-Allow-Headers'] = 'X-Requested-With, X-Prototype-Version, Token'
      headers['Access-Control-Max-Age'] = '1728000'

      render :text => '', :content_type => 'text/plain'
    end
  end
end
```

至此，数据已经可以从http://votenotes.com/ 提交到 http://localhost:3000/sub_emails了，实现了跨站数据提交的功能。

>  特别注意：  

以上代码中Api接口关闭了CSRF Token验证，所以很容易被人CSRF攻击。解决办法可以尝试采用约定协议通信，并对通信凭证签名加密的方式。不过由于本应用本身对数据要求不是很敏感，所以暂时不考虑了。


>  延伸思考：

以上实现方式，假如在完全解决了安全问题的情况下， 是否可以做到前后端分离呢？从而开发网站也就像做App一样，后端统一为一套Api接口了。
