---
title: How to use cancan in rails model?
date: 2014-06-29 14:38:38 +0800
tags: [ruby]
---


There's a wiki entry at github for this: [http://wiki.github.com/ryanb/cancan/ability-for-other-users](http://wiki.github.com/ryanb/cancan/ability-for-other-users)

You need to change your model:

```
class User < ActiveRecord::Base  
  def ability
    @ability ||= Ability.new(self)  
  end  

  delegate :can?, :cannot?, :to => :ability
end
```

Then you can check abilities like this:

```
user.can?(:update,@article)
```
