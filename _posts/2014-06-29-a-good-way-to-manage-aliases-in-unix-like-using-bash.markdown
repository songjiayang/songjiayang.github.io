---
title: A good way to manage aliases in Unix-like using BASH
date: 2014-06-29 14:19:11 +0800
tags: [linux]
---

To introduce how it works, a example [*bundle exec rake* ] alias will be given.

Now, let's start.

1.1 open your ~/.bashrc file

```
vim ~/.bashrc
```

1.2 add alias commands trigger commands to ~/.bashrc

```
# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```

1.3 add all the alias commands that you want to ~/.bash_aliases  

```
alias rake='bundle exec rake'
```

> All alias commands with the same format like  `alias <name>='<linux_command>'`
