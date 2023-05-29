---
title: Ordering null values last in thinking-sphinx
date: 2014-04-02 15:12:38 +0800
tags: [ruby]
---

Sometimes , you want to order the records by some columns that should not be nil. that is if the ordered columns values is nil the record should list the last .

### How to deal with it by think-sphinx? ###

#### Step 1: Add column nil index

```
has "example IS NULL", :as => :example_nil, :type => :boolean
```

#### Step 2: Order by *example_nil*

```
sphinx_options[:order] = "example_nil  ASC, xxx"
```

References:  
1. [https://github.com/pat/thinking-sphinx/issues/737](https://github.com/pat/thinking-sphinx/issues/737)
