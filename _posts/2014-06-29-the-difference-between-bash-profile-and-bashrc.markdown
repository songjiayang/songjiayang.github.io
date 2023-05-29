---
title: The difference between .bash_profile and .bashrc
date: 2014-06-29 14:43:23 +0800
tags: [linux]
---


Traditionally, when you log into a Unix system, the system would start one program for you. That program is a shell, i.e., a program designed to start other programs. It's a command line shell: you start another program by typing its name. The default shell, a Bourne shell, reads commands from `~/.profile` when it is invoked as the login shell.

Bash is a Bourne-like shell. It reads commands from `~/.bash_profile` when it is invoked as the login shell, and if that file doesn't exist¹, it tries reading `~/.profile` instead.

You can invoke a shell directly at any time, for example by launching a terminal emulator inside a GUI environment. If the shell is not a login shell, it doesn't read `~/.profile`. When you start bash as an interactive shell (i.e., not to run a script), it reads `~/.bashrc` (except when invoked as a login shell, then it only reads `~/.bash_profile` or `~/.profile`.  

Therefore:
1. `~/.profile` is the place to put stuff that applies to your whole session, such as programs that you want to start when you log in (but not graphical programs, they go into a different file), and environment variable definitions.
2. `~/.bashrc` is the place to put stuff that applies only to bash itself, such as alias and function definitions, shell options, and prompt settings. (You could also put key bindings there, but for bash they normally go into `~/.inputrc`.)
3. `~/.bash_profile` can be used instead of `~/.profile`, but it is read by bash only, not by any other shell. (This is mostly a concern if you want your initialization files to work on multiple machines and your login shell isn't bash on all of them.) This is a logical place to include `~/.bashrc` if the shell is interactive. I recommend the following contents in `~/.bash_profile`:

```
if [ -r ~/.profile ]; then . ~/.profile; ficase "$-" in *i*) if [ -r ~/.bashrc ]; then . ~/.bashrc; fi;; esac
```
On modern unices, there's an added complication related to ~/.profile. If you log in in a graphical environment (that is, if the program where you type your password is running in graphics mode), you don't automatically get a login shell that reads ~/.profile. Depending on the graphical login program, on the window manager or desktop environment you run afterwards, and on how your distribution configured these programs, your ~/.profile may or may not be read. If it's not, there's usually another place where you can define environment variables and programs to launch when you log in, but there is unfortunately no standard location.

Note that you may see here and there recommendations to either put environment variable definitions in~/.bashrc or always launch login shells in terminals. Both are bad ideas. The most common problem with either of these ideas is that your environment variables will only be set in programs launched via the terminal, not in programs started directly with an icon or menu or keyboard shortcut.

References:  
1. [http://superuser.com/questions/183870/difference-between-bashrc-and-bash-profile](http://superuser.com/questions/183870/difference-between-bashrc-and-bash-profile)
