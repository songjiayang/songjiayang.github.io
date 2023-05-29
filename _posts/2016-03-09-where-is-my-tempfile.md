---
title: "where is my tempfile?"
date: 2016-03-09 22:31:00 +0800
tags: [ruby]
---
![tempfile.png](/images/tempfile.png)

The benefits of `Tempfile` are:

1. Automatically deleted.
2. Unique file name.
3. Nice place (`/var`) stored at disk.

What's my problem? Let me show you my code:

```
def qr_file(url)
    temp = Tempfile.new(['qr', '.png'])
    path = temp.path
    qrcode = RQRCode::QRCode.new(url)
    qrcode.as_png(
      resize_gte_to: false,
      resize_exactly_to: false,
      fill: 'white',
      color: 'black',
      size: 200,
      border_modules: 0,
      module_px_size: 0
      file: path
    )
    path
end

def build_qr_cell(url, opts={})
  @excel.add_image(image_src: qr_file(url), noSelect: false, noMove: false) do |image|
    image.width = 200
    image.height = 200
    image.start_at opts[:column_index], opts[:row_index]+1
  end
end

```

There are two methods `qr_file` and `build_qr_cell` . The first one use to create a tempfile and return file path, the other one will use the path.

When I execute my code, I get **file not exist?** error sometimes. Yes, it is sometimes rather than always.

**Code is logic, not accident**.

It's so strange that I want to figure out the reason.

I review [Tempfile](http://ruby-doc.org/stdlib-2.3.0/libdoc/tempfile/rdoc/Tempfile.html) doc, stop eyes at this line.

> When a Tempfile object is garbage collected, or when the Ruby interpreter exits, its associated temporary file is automatically deleted.

Oh, Maybe I get it. I guess the reason is Ruby GC.

When after `qr_file` method call, the local variable `temp` lost its context, it should be garbage collected.

As you known, Ruby execute GC has time limit. Sometimes you use the file, GC doesn't excute, but sometimes had. when GC executed before you use it, the file should be deleted and raise **not exist** error.  

How to prove my guess? I think execute GC by hand is one way. to see:

```
#in irb, with ruby 2.3
require 'tempfile'
def generate_tempfile
  temp = Tempfile.new('')
  temp.path
end
file = generate_tempfile
File.exist? file # most time is true
GC.start # execute gc by hand
File.exist? file # should be false
```

I test my code in `irb`, I get file deleted every time after execute GC by hand. Cool, this is answer as my expect.

*When I test the code, I find it doesn't work at lower than the 2.3 version ruby. I guess in ruby 2.3, the GC has improved for this case. so I think 2.3 is the most fast ruby version so far.*

How to fix my code ?   

1. Use tempfile in one function or return the temp object.
2. Use `File` and `tmpdir` std lib to replace `Tempfile`, and use system cron job for delete stuff.


Conclusion:   

1. Don't use tempfile cross method.  
2. Ruby 2.3 is the best version so far, please use it relaxed。

This is a wonderful experience， I'm really interested in  Ruby `2.3` and `GC` now, Maybe I will write some articles about them  later.

Await please!  
