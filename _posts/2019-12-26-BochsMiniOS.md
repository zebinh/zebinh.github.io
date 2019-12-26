---
layout: post
title: bochs虚拟机的安装
date: 2019-12-26
tags: 操作系统
---

# bochs虚拟机的安装 - 动手写一个最小的操作系统环境搭建

最近在研究操作系统，读了理论的书籍后，打算自己实现一遍，加深理解。因此选了于渊老师的书《Orange'S一个操作系统的实现》跟着书实战一遍。在安装bochs虚拟机过程中遇到一些问题并已解决，特记录在此。

我的环境：
+ 宿主机为win10
+ 虚拟机为vagrant(底层是virtualbox)，这里使用virtualbox，vmware都是可以的
+ 虚拟机中的系统为CentOS 7，命令行版，无界面。当然，这里如果你是有界面的linux发现版更好。
+ win上的命令行终端使用的是git bash，这里使用其他的如xshell或者secureCRT也可以

bochs要启动调试功能的话不能直接使用yum安装，只能下载源码包编译安装了。

## 安装编译环境和依赖包

``` sh
yum install -y gcc glibc-headers gcc-c++ libXrandr-devel
```

## 下载并安装bochs

现在时间是2019年12月26日，输入如下命令下载2.6.9版本的即可
``` sh
# 下载
curl -O https://nchc.dl.sourceforge.net/project/bochs/bochs/2.6.9/bochs-2.6.9.tar.gz

# 解压
tar -zxvf bochs-2.6.9.tar.gz
```
cd进入解压后的目录并配置编译选项
``` sh
# cd进入目录
cd bochs-2.6.9

# 配置编译选项
./configure --with-x11 --with-wx --enable-debugger --enable-disasm --enable-all-optimizations --enable-readline --enable-long-phy-address --enable-ltdl-install --enable-idle-hack --enable-plugins --enable-a20-pin --enable-x86-64 --enable-smp --enable-cpu-level=6 --enable-large-ramfile --enable-repeat-speedups --enable-fast-function-calls --enable-handlers-chaining --enable-trace-linking --enable-configurable-msrs --enable-show-ips --enable-cpp --enable-debugger-gui --enable-iodebug --enable-logging --enable-assert-checks --enable-fpu --enable-vmx=2 --enable-svm --enable-3dnow --enable-alignment-check --enable-monitor-mwait --enable-avx --enable-evex --enable-x86-debugger --enable-pci --enable-usb --enable-voodoo
```
输入make命令编译。
```
make
```

## 报错解决
报错如下：
```
make: *** No rule to make target 'misc/bximage.cc', needed by 'misc/bximage.o'.  Stop.
```
解决方案为：
```
cp misc/bximage.cpp misc/bximage.cc
```
接下来还有类似的No rule to make target 'xxxx'，解决方案类似。
```
cp iodev/hdimage/hdimage.cpp iodev/hdimage/http://hdimage.cc
cp iodev/hdimage/vmware3.cpp iodev/hdimage/http://vmware3.cc
cp iodev/hdimage/vmware4.cpp iodev/hdimage/http://vmware4.cc
cp iodev/hdimage/vpc-img.cpp iodev/hdimage/http://vpc-img.cc
cp iodev/hdimage/vbox.cpp iodev/hdimage/http://vbox.cc
```
make执行成功后，安装。
```sh
make install
```

## bochs配置文件

要启动bochs运行我们编写的操作系统，需要先配置bochs。bochs解压目录bochs-2.6.9下，复制一份配置文件，
``` sh
# cp后的第一个参数有点号.
cp .bochsrc bochsrc
```
这里假设你已经写好了汇编代码并写入软盘镜像boot.img了。vi bochsrc打开文件按如下步骤修改。
``` sh
# romimage: file=$BXSHARE/BIOS-bochs-latest, options=fastboot改为如下，其中$BXSHARE改为你的解压目录下+bios
romimage: file=/usr/local/bochs-2.6.9/bios/BIOS-bochs-latest, options=fastboot
# 同上
vgaromimage: file=/usr/local/bochs-2.6.9/bios/VGABIOS-lgpl-latest

# floppya: 1_44=/dev/fd0, status=inserted这行改为你的软盘镜像，我的是boot.img
floppya: 1_44=boot.img, status=inserted

#ata0-master: type=disk, mode=flat, path="30M.sample"这行要加注释注释掉

# 启动盘修改，改为软盘启动
boot: floppy
#boot: disk

#sound: driver=default, waveout=/dev/dsp. wavein=, midiout=注释掉这行声音配置

```
OK，以上配置完毕，使用如下命令启动bochs
```
sudo ./bochs -f bochsrc
```
注意这里要使用sudo权限执行，否者会报no bootable device错误。
## bochs启动报错解决

```
Message: ata0-0: could not open hard drive image file '30M.sample'
```
上面这个报错，是因为引导设备默认为硬盘，这里应该换为我们写的boot.img软盘。注释掉下面硬盘这行，打开软盘这行。
```
boot: floppy
#boot: disk
```

---


```
Bochs is not compiled with lowlevel sound support.
```
注释掉配置文件中这行关于声音设备的即可。
```
sound: driver=default, waveout=/dev/dsp. wavein=, midiout=
```

----

下面这个报错如果你使用的是命令行连接虚拟机的方式，那么就会有这个报错。

```
00000000000p[GUI   ] >>PANIC<< Cannot connect to X display
========================================================================
Event type: PANIC
Device: [GUI   ]
Message: Cannot connect to X display

```
如下图：
![20191226205936.png](https://i.loli.net/2019/12/26/dOKhRbuMy7GEQUI.png)

这个问题的原因是，我是使用git bash命令行连接虚拟机里的CentOS 7，而bochs是有界面的，图像界面无法在git bash命令行下展示。
解决方案有两种：
+ 1. 你在图形界面下的linux安装bochs，这种当然不符合我们的需求，因为我们已经在命令行下安装了bochs，在安装图形界面版的linux太麻烦。
+ 2. 在linux中安装X11，win使用vnc viewer连接到linux上，vnc viewer就可以显示图形界面啦。

我们使用第二种，安装vnc viewer连接linux显示图形界面，见我的另一篇文章，[VNC Viewer登录Linux(centos7)可视化界面
](https://juejin.im/post/5d81a903f265da03ed1987ba)

-----
```
no bootable device
```
如果确认自己的引导设备已经配置为软盘并且没有配置错误，那就是你执行./bochs -f bochsrc没有加sudo，加上sudo执行即可。

----

最终运行起来的界面如下：
![20191226231401.png](https://i.loli.net/2019/12/26/7A9KzTiWOrRvmcM.png)

刚执行完sudo ./bochs -f bochsrc启动bochs时并没有马上显示我们的Hello OS World！，因为bochs启动debug模式了，如下图。

![20191226225725.png](https://i.loli.net/2019/12/26/Wc5AGPlbIfOXarz.png)
这时我们需要切到shell终端界面，终端显示的是<bochs:1>，我们输入c回车，继续让bochs运行即可。

最后，如果觉得卡的话，调大虚拟机内存。