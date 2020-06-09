---
title: "Strong parameters: allow hashes with unknown keys to be permitted"
date: 2015-08-25 23:31:00 +0800
tags: [ruby]
---

Rails 4 使用 StrongParameter 来处理用户数据白名单, 日常开发中有时会遇到这样一个问题，即嵌套的参数是一个 hash, 并且这个 hash 的 keys 是不确定的（用户可以任意输入）。

举例说明：

Contact 包含了 company 信息， company 是个 store， 其 attributes 可以自定义的，
用户提交的数据类似这样：

```
 {contact: {name: 'User Name', company: {c_k1: c_v1, c_k2: c_v2} }}
```

这个 case 的 permit 该怎么写？

其实很简单，我们可以使用 tap 方法：

```
def contact_params
  params.require(:contact).permit(:name).tap do |whitelisted|
    whitelisted[:company] = params[:contact][:company]
  end
end
```

参考资料： [https://github.com/rails/rails/issues/9454#issuecomment-14167664](https://github.com/rails/rails/issues/9454#issuecomment-14167664)
