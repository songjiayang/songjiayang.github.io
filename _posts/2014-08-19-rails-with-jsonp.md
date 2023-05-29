---
title: Rails with JSONP
date: 2014-08-19 22:31:00 +0800
tags: [ruby]
---


今天遇到一个AJAX跨站请求api的问题，出现问题的本质是浏览器出于安全原因，限制了Ajax跨域请求。那如何处理这个问题？经过搜索之后，决定采用JSONP的方式。如果你还不太了解JSONP，请[点击这里](http://zh.wikipedia.org/wiki/JSONP)。

#### JSONP在Rails中的具体使用：

1、 在子站的json请求中添加callback参数,并指定请求数据类型为jsonp。

```
  params = {dataType: 'jsonp'}
  url = "http://server2.com/api/search.json?callback=?"
  $.getJSON(url, params).success (response) =>
    #console.log response
```

2、修改主站后端代码，使其支持JSONP.

```
  format.json {
    items = [ {id: 1, name: 'hello' } ]
    if params[:callback]
      render :json => { items: items }, :callback => params[:callback]
    else
      render :json => { items: items }
    end
  }

```
至此，我们已经能够从主站拿到api数据了,从而解决了AJAX跨域的问题。

> 注意：JSONP只能用于get请求，我们不能为了方便，使用JSONP来处理本该post的数据操作，这将带来很大的安全隐患。
