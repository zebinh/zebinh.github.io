---
layout: post
title: git bash安装tmux实现多标签
date: 2020-04-09
tags: 工具
---

# git bash安装tmux实现多标签

众所周知，git bash下是不能实现多标签页的，每次使用git bash的ssh连接多台机器时，需要打开多个git bash程序。而tmux能解决这个问题，并且tmux比这个还要强大。

tmux是一个终端复用器(terminal mutilplexer)。何谓终端复用器呢？平时我们的使用git bash终端通过ssh连接到远程之后，会话就开始了，当关闭终端时，会话就结束，远程正在执行的任务也会结束，即会话和终端窗口是绑定在一起的。tmux就是为了解决这个问题，让窗口和会话解绑。

## git bash安装tmux

git bash中执行以下命令，即可安装tmux。如果以下操作完tmux没法使用，注意git升级到最新版，我遇到的坑就是git2.9版本安装完tmux后打tmux命令没反应。升级完git2.26后即可。

```bash
git clone https://github.com/xnng/bash.git
cd bash
cp tmux/bin/* /usr/bin
cp tmux/share/* /usr/share -r
```

新建tmux配置文件，
```
vi ~/.tmux.conf
```

原因复制如下配置到上面的配置文件中即可。

```conf
setw -g mouse
set-option -g history-limit 20000
set-option -g mouse on
bind -n WheelUpPane select-pane -t= \; copy-mode -e \; send-keys -M
bind -n WheelDownPane select-pane -t= \; send-keys -M
```

## tmux简单操作

+ 新建一个会话并命名为work：tmux new -s work
+ 这时打开了一个tmux会话，窗口底部是一个绿色的信息显示条
+ 输入tmux detach命令，分离窗口和会话，这是你会退出tmux会话，回到git bash终端
+ 输入tmux ls，你会看到自己后台正在跑的tmux会话，这是你关闭git bash终端，会话也不会关闭
+ 此时可以输入tmux new -s mytest新建一个新会话，相当于开了两个窗口了。
+ 使用tmux switch -t work可以切换到work会话

以上，还有许多的快捷键可以使用，如，在tmux会话中，按ctrl+b一下，再按以下w键，会弹出所有窗口的列表，此时你选择一个窗口即可，切换非常方便。

更多的快捷键请自行网络搜索。

## tmux介绍

tmux的层次划分为：一个session下有多个window，一个window下有多个pane(面板)
+ 新建session：tmux new -s {sessionName}，直接输入tmux也可新建一个自动命名的session
+ 新建window：tmux new-window -n {windowName}
+ 新建pane：tmux split-window(划分上下两个窗格)，tmux split-window -h(划分左右两个窗格)
+ 快捷键：显示所有window：```C-b w```表示先按一次ctrl-b，再按一次w

## tmux内容复制

按住shift选择内容后按右键选复制，记住是**按住**shift键。粘贴时可以按住shift键点击右键，再选择粘贴。

## tmux常用快捷键

tmux的快捷键都有前缀键，默认为ctrl+b，记为```C-b```，需先按前缀键再按指定的键，如```C-b w```打开window列表，表示按前缀键后按w，依此类推。一般的操作流程和快捷键如下：

+ 查看帮助：C-b ?
+ 新建session：建议还是使用命令行比较好，因为可以命名session：tmux new -s {sessionName}
+ 断开/分离当前会话：C-b d
+ 新建窗口：C-b c，或命名tmux new-window -n {windowName}
+ 打开窗口列表用于切换窗口：C-b w
+ 关闭窗口：C-b &
+ 垂直分割窗格：C-b %
+ 关闭窗格：C-b x