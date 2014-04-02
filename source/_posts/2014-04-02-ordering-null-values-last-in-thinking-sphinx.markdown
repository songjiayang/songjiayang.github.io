---
layout: post
title: "Ordering null values last in thinking-sphinx"
date: 2014-04-02 15:12:38 +0800
comments: true
categories: [think-shpinx, order]
---

Sometimes , you want to order the records by some columns that should not be nil. that is if the ordered columns values is nil the record should list the last .


** How to deal with it by think-sphinx? **

add column nil index

``` ruby
has "example IS NULL", :as => :example_nil, :type => :boolean
```

order by `example_nil`

``` ruby
sphinx_options[:order] = "example_nil  ASC, xxx"
```


https://github.com/pat/thinking-sphinx/issues/737


