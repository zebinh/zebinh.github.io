---
layout: post
title: Windows下的极简主义工作法
date: 2019-12-14
tags: 生活
---
# Windows下的极简主义工作法

本文对于程序员效果更佳，当然其他同学也可以捡一些小工具提高生产效率。

作为一个完美主义者，我喜欢干净的Windows，不喜欢一些杂七杂八的软件在我的系统上乱跑。同时，平时也喜欢搜索各种小工具提高电脑办公的生产效率。最近混迹各种论坛，搜集到了一些有用的小工具。能最大程度的简化开发和保存Windows上的简洁。以下，我推荐下自己总结的这些小工具。

假设现在系统是很干净的，刚刚重装完成没有杂七杂八的软件。

## 划分D盘作为非系统盘

这里我建议磁盘只划分为C盘和D盘两个盘，C盘作为系统盘，单独存放系统，D盘作为文件盘，存放文件。这样以后重装系统的话可以格式化C盘而不影响你的文件，即文件不会丢失。另外，也不建议划分多个盘，如E盘F盘等，这样每次查找文件就方便很多，不用想到底我的文件是放在D盘E盘还是F盘，少了一步思考的步骤。

## Scoop包管理软件

刚装完系统，当然最主要的工作就是要安装各种软件了。但是软件太多了，安装起来很麻烦。有没有工具可以只输入一条命令，就能帮我们成套安装软件并配置好环境呢，因为实在不想再去折腾像JDK这样的环境变量。这里我就要推荐Scoop了，他只需要输入一条命令就可以帮你安装软件，并解决环境变量的配置问题。

安装Scoop步骤如下，这里Scoop我建议安装到D盘，打开PowerShell，按顺序复制并粘贴以下命令即可。
``` powershell
set-executionpolicy remotesigned -scope currentuser
[environment]::setEnvironmentVariable('SCOOP','D:\Scoop','User') # 这里设置安装到D盘
$env:SCOOP='D:\Scoop' # 这里设置安装到D盘
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
```
如果没报错就表示安装成功了，很方便吧。复制以下命令并粘贴，为Scoop添加官方拓展仓库，可以下载更多软件。
``` powershell
scoop bucket add extras
scoop bucket add versions
```
至此，Scoop安装盘配置完毕，可以通过以下命令搜索和安装你想要的软件
``` powershell
# 搜索软件
scoop search 软件名称
# 安装软件
scoop install 软件名称
# 卸载软件
scoop uninstall 软件名称
```

## 软件推荐

以下推荐一些我我觉得很好的软件。有些不能使用scoop安装。

+ utools：小工具的合集，可以快捷记录你的待办事项todos，压缩图片，修改hosts文件(修改hosts配合虚拟机使用很完美)，快捷搜索本地文件(取代listary和everything)。
+ snipaste：截图和贴图工具，同时还可以屏幕取色
+ deskpins：桌面置顶工具
+ win+v：windows自带的剪贴板历史工具
+ vagrant：程序员必备的虚拟机编排软件，快捷安装配置虚拟机，比单纯使用virtualbox好很多，当然vagrant是使用virtualbox创建虚拟机的，所以安装vagrant同时也要安装virtualbox
