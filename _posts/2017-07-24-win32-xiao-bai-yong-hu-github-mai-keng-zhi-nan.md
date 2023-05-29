---
title: "win32 小白用户 GitHub 埋坑指南"
date: 2017-07-24 22:02:52
tags: [git]
---

这周作为教练参加了上海举办的 Girl's Coding Day 活动，期间发现有部分学员使用的是 win 32位的操作系统，出现了按照教程无法安装 GitHub Desktop 的问题 ，故编写了本篇指导，帮助那些对  Git,  GitHub 不是很了解的同学，方便其快速上手。

本篇指导主要运行在 win32 操作系统上。

### 概念

Git: 是一个分布式的代码管理系统，使用它可以很方便的做到多人协作，同步，回滚代码，而使用频率较高的命令包括：

- `git  init` （初始化项目）
- `git  add .` （添加修改文件)
- `git commit -m "xxx"` （提交修改到本地仓库)
- `git push  origin xxxx` （提交代码到中央仓库）
- `git pull  origin xxxx` （从中央仓库下载代码）

GitHub: 是一个git 代码在线托管网站，主要实现了使用浏览器查看代码，分享和管理代码，简单理解就是代码中央仓库的可视化管理平台

### 使用步骤

Step 1: 访问 https://pan.baidu.com/s/1dFfAzyL ，下载安装 win32 版本 git。

- 安装完成后，点击开始菜单 ->  输入 git -> 点击 Git CMD  进入 git 终端。
- 在终端光标处输入 `git config  --global user.name xxxx` 设置账号名称。
- 在终端光标处输入 `git config  --global user.email xxxx`  设置账号 email。

![pic1.png](/images/github/pic1.png)

Step 2: 访问 https://github.com/new ， 创建任意项目，如图

![pic2.png](/images/github/pic2.png)

Step3: 使用 git https 管理管理项目

- 如图拷贝项目地址

![pic3.png](/images/github/pic3.png)

- 使用 git clone 拉取代码

1. 点击开始菜单 -> 输入  git -> 点击 Git CMD  进入 git 终端。
2. 在终端光标处输入 cd  Desktop , 将工作目录切换到电脑桌面。
3. 在终端光标处 `git clone  <项目链接>` ，比如我这里的 git clone https://github.com/songjiayang/GirlsCodingDay.git ，此时你将看到桌面会产生项目目录文件，表示项目已从 GitHub 拉取成功。

![pic4.png](/images/github/pic4.png)

- 修改并提交代码

1. 使用 atom 编辑器，打开此项目，开始编写代码（可以考虑使用模版，将模版的内容全部拷贝到这个项目目录下）
2. 代码修改完后，执行 git add . 添加所有修改的文件。
3. 执行 `git commit -m "add xxxx"` 提交修改文件到本地。
4. 执行 `git push origin master`，提交代码到 GitHub, 这一步会提示输入 GitHub 账号。 

![pic5.png](/images/github/pic5.png)

Step4:  设置 GitHub Pages

- 访问项目的  GitHub 页面设置页面，比如我的， https://github.com/songjiayang/GirlsCodingDay/settings 。
- 修改 GitHub Pages 选择，并保存。
- 使用自动生成的 GitHub Pages 地址即可访问自己的网站了。

### 写在最后

学员对于这次活动这样说：

> 以前看到这些东西都是一堆乱码，而现在发现它们都是有逻辑的。

我想这就是最大收获吧，敢于挑战自我，不断学习，进步的人都是可敬的！


