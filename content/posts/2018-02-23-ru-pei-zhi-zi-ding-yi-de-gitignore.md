---
title: ".gitignore 瘦身秘诀"
date: 2018-02-22 16:00:25
tags: [git]
---

我们在使用 `git` 的时候经常会遇到这样一个问题，团队成员使用的编辑器很多，比如 `vscode, vim, idea` ，而它们产生的临时文件又各不相同，这时我们该怎么处理？

### 常见方法

将用到的编辑器产生的临时文件格式都添加到 .gitignore 文件中，所以最后它就变成这样或者更多：

```
# Editor files #	
################	
*~	
.*.swp	
.*.swo	
*.iml	
.idea	
tags	
.history	
.vscode
```

这样的问题：

- 需要修改 .gitignore，添加新的文件格式，然后 git commit。
- 编辑器很多而且有新的产生，何时是个头。

### 改进方法

其实在项目目录下，有个叫做 `.git/info/exclude` 的文件，它可以用于自定义个人忽略选项，从而避免以上问题。

以 vscode 编辑器为例，我们做以下简单调整即可：

- 删除 `.gitignore` 中 Editor files 有关内容。
- 在 `.git/info/exclude` 中添加 `.vscode`。