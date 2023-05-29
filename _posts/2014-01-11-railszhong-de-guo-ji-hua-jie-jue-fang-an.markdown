---
title: Rails 中的国际化解决方案
date: 2014-01-11 01:41:39 +0800
tags: [ruby] 
---

工作经常会在rails中遇到I18n的相关问题，目前为止，我所使用过的方案有两种，各自有各自的好处。

方案1: 使用rails官方自带的I18n，文档链接[http://guides.rubyonrails.org/i18n.html](http://guides.rubyonrails.org/i18n.html)

此方法主要使用了I18n这个Module,而I18n最为重要的两个方法是 **translate**，**localize**
他们的使用十分简单，如下：

```
I18n.t 'hello'  => "translation missing: xxx"
I18n.l Time.now   => "Tue, 12 Nov 2013 05:56:47 pm CST"
```

日常国际化做翻译的时候，需要用到**t**这个方法，如上。这个方法可以在直接使用，而不需要I18n来call.如果你的I18n.locale 没有对应的翻译，那么系统将显示 “translating missing: xxx”的字样。既然这样，你需要做的就是，将对应的翻译的字表（yml文件）事先写好，这样rails就会根据这个字表来做翻译。

具体操作：

配置翻译文件的路径

```
# in config/initializers/locale.rb

# tell the I18n library where to find your translations
I18n.load_path += Dir[Rails.root.join('lib', 'locale', '*.{rb,yml}')]

# set default locale to something other than :en
I18n.default_locale = :pt
```


在对应配置路径下面补充对应的yml文件
```
# config/locales/zh.yml
zh-CN:
  hello: 你好
```
通过以上设置，然后通过I18n.locale="zh-CN" 将本地语言环境切换到中文环境，然后再调用t('hello'),将显示“你好”。

采用官方自带的I18n，无需依赖其他gem, 从而代码维护成本低，版本依赖小，上手更容易，代码容易维护。不过这种方式的缺点也很明显，需要书写大量的yml,这样对于yml文件中的key的命名就很头痛，既要满足hash key的命名语法，又要解决翻译过多带来的命名冲突，所以无奈会引入命名空间，导致yml中key的结构层数太多，会出现很多冗余的翻译。



#### 方案2: 使用gettext来做翻译

1） 添加gem依赖

```
#I18n support
gem 'i18n'
gem 'rails-i18n'
gem 'gettext',  :require => false, :group => :development
gem "gettext_i18n_rails"
gem 'gettext_i18n_rails_js', github: 'songjiayang/gettext_i18n_rails_js'
gem 'ruby_parser', :require => false, :group => :development
```

2）执行bundle install

3)  在项目下建立一个locale/zh_CN的文件夹，其中zh_CN可以叫任意名字，它表示你要翻译的目标语言。

4)  执行 `rake -T ` 查看相关命令，命令包括

```
rake gettext:add_language LANGUAGE=xx
rake gettext:find                    # Update pot/po files
rake gettext:pack                    # Create mo-files for L10n
rake gettext:po_to_json              # Convert PO files to js files in app/assets/locales
rake gettext:store_model_attributes
```

5)  新建config/initializers/fast_gettext.rb 文件，并配置如下

```
I18n.default_locale = :en
FastGettext.add_text_domain 'app', :path => 'locale', :type => :po,
                               :ignore_fuzzy => true, :report_warning => false
FastGettext.default_available_locales = ['en_US','zh_CN']  #all you want to allow
FastGettext.default_text_domain = 'app'
```

6) 执行 `rake gettext:po_to_json`，此命令会新建app/assets/javascripts/locale/<lang>/app.js文件。

7）在application.js中引入gettext
```
//= require_tree ./locale
//= require gettext/all
```

8) 项目中添加I18n切换的操作,例如
```
class ApplicationController < ActionController::Base

  protect_from_forgery with: :exception
  before_filter :set_gettext_locale

  protected
    def set_gettext_locale
      requested_locale = params[:locale] || session[:locale] || request.env['HTTP_ACCEPT_LANGUAGE'] || I18n.default_locale
      requested_locale = 'zh_CN' if %w(zh zh-tw zh_tw).include?(requested_locale.downcase)
      requested_locale = 'en_US' if %w(en en-gb en_gb).include?(requested_locale.downcase)
      locale = FastGettext.set_locale(requested_locale)
      I18n.locale = locale
      puts I18n.locale
      session[:locale] = I18n.locale.to_s
    end
end
```

9)  在页面中添加locale，例如在application.html.erb中添加
```
<html lang = <%= "#{I18n.locale}"%>>
```

10)  添加需要翻译的语句，并执行翻译

```
<%=_('hello')%>
```


```
console.log(__("hello"))
```

执行 `rake gettext:find` 分析需要翻译的语句，然后到locale/xxx/app.po文件中去执行翻译即可，翻译完毕后，需要执行`rake gettext:po_to_json` 将翻译也复制一份到js当中去。

总结：至此大工告成，可以浏览器中访问 `http://localhost:3000/?locale=zh` 测试了，这里有一个我写的例子[rails_i18n_example](https://github.com/songjiayang/rails_i18n_example)

参考资料：  
1. [https://github.com/grosser/gettext_i18n_rails](https://github.com/grosser/gettext_i18n_rails)  
2. [https://github.com/nubis/gettext_i18n_rails_js](https://github.com/nubis/gettext_i18n_rails_js)
