---
layout: post
title: windows下的Linux环境（Cygwin同类对比）
date: 2020-04-06
tags: 工具
---

# windows下的Linux环境（Cygwin同类对比）

因为平时在win下做开发，但已经习惯了使用linux下一些方便的命令，并且git基本上每个程序员都必备的，所以现在基本使用git bash终端来代替cmd终端了。所以想一探git bash的究竟。

以前有了解过Cygwin，也安装过cmder这种终端工具，但速度都没有git bash快，因此就一直使用git bash了。

## POSIX

想来讲讲为什么会有Cygwin这种工具。现在主流的操作系统是微软的Windows和类unix系统，如果各系统平台都能提供相同的系统函数库，开发者在这个系统函数库基础之上编写软件代码，那么就很容易将软件移植到各个系统平台。这就是POSIX的初衷。

类unix系统是兼容POSIX的。但windows却支持得不好。

所以很多能运行在linux系统下的程序不能在windows下运行，因此需要有一种运行在windows下的程序，能构造一种POSIX环境，这样在windows下就能跑linux下的程序了。

## Cygwin

Cygwin是一个可原生运行于Windows系统上的POSXI兼容环境。

> All problems in computer science can be solved by another level of indirection
> （计算机科学领域的任何问题都可以通过增加一个间接的中间层来解决）。  -- David Wheeler

Cygwin就是在Windows中增加了一个中间层。

从Cygwin官网上看，它包含两个功能：
+ 是GUN和开源工具的大集合
+ 提供POSIX API

![GrDnQ1.png](https://s1.ax1x.com/2020/04/05/GrDnQ1.png)

所以，理论上运行在linux下的程序只要经过Cygwin重新编译，就可运行在Windows下。同时Cygwin还提供了Linux下的很多命令，如ls、ps等。


## MinGW

MinGW与Cygwin皆可用来移植Unix软件到 Windows，但它们采用截然不同的实现。Cygwin 旨在提供一个完整的 POSIX 层，包括主流 Unix 的系统调用及库实现；其重视兼容性优先于性能。相对的，MinGW则着重简化与性能。因此，它并不提供某些难以用Windows API实现的POSIX API。

可以简单的认为，
+ Cygwin编译的是源码，然后使用一个cygwin1.dll中间层转化POSIX调用
+ MinGW编译前将POSIX调用的代码“翻译”成win32 api能理解的代码，然后再调用windows api进行编译。

## MSYS

上面的MinGW只是一个Windows版本的开发套件，可以做代码“翻译”，它不是一个虚拟的类Unix系统。所以在MinGW中你并不能执行类似ls、ps这种命令，这时，需要MSYS结合MinGW来使用。MSYS是一个Windows平台上运行的类Unix模拟环境。

## git bash

git bash使用的就是MinGW


## winpty

在git bash中如果想使用windows宿主中的命令，可以使用winpty连接到win系统，winpty相当于ssh登录到了win系统中。如，你要执行win中的tree命令，可以使用
```
winpty tree.com
winpyt dir
```
即，使用winpty连接到win中执行命令