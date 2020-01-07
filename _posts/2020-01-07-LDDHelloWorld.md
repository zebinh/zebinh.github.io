---
layout: post
title: CentOS7编写HelloWorld驱动程序入门(内核模块)
date: 2020-01-07
tags: 读书
---

# CentOS7编写HelloWorld驱动程序入门(内核模块)

最近在看LDD(Linux设备驱动程序)这本书，第二章写的一个简单的HelloWorld模块，没想到有很多坑，这里记录一下，方便后来人脱坑。

![lyLLyF.png](https://s2.ax1x.com/2020/01/07/lyLLyF.png)

## 环境

+ linux内核：3.10.0-1062.9.1.el7.x86_64（使用命令uname -r查看），即3.10.0版本
+ CentOS版本：CentOS7.7.1908（使用命令cat /etc/redhat-release命令查看）

## 编写模块代码

照着书上写的HelloWorld模块代码如下，
```c
#include <linux/init.h>
#include <linux/module.h>

MODULE_LICENSE("Dual BSD/GPL");

static int hello_init(void){
        printk(KERN_EMERG "Hello, world\n");
        return 0;
}

static void hello_exit(void){
        printk(KERN_EMERG "Goodbye, cruel world\n");
}

module_init(hello_init);
module_exit(hello_exit);
```
以上源码命名为hello.c，之后执行```gcc hello.c```命令报错，
> linux/init.h: No such file or directory

因为看到```#include <linux/init.h>```就想到貌似我的系统没有linux/init.h这个头文件，遂网上搜索发现这个头文件是linux内核的头文件，位于/usr/src/linux/include目录下，按路径在本机查找，发现我并没有/usr/src/linux目录。

因此下一步我们要安装linux内核源码。

## 安装Linux内核源码

首先上官网（http://vault.centos.org）下载自己系统对应的源码包，以我的CentOS7.7为例，路径为(http://vault.centos.org/7.7.1908/updates/Source/SPackages/kernel-3.10.0-1062.9.1.el7.src.rpm)。

国外路径下载较慢，建议使用迅雷下载再上传到CentOS虚拟机中。

下载完成后得到kernel-3.10.0-1062.9.1.el7.src.rpm包，假设位于当前home(路径为~)目录下，执行
```sh
rpm -ivh kernel-3.10.0-1062.9.1.el7.src.rpm
```
得到rpmbuild目录，之后执行如下命令安装源码包。
```sh
# 安装源码包
rpmbuild -bp --target=$(uname -m) kernel.spec
# 如果包rpmbuild命令未找到，先安装，yum install rpm-build

```

执行后可能会报错确实编译工具，则安装对应工具即可，整理如下。
```
sudo yum install -y kernel-devel rpm-build redhat-rpm-config asciidoc hmaccalc perl-ExtUtils-Embed pesign xmlto audit-libs-devel binutils-devel elfutils-devel elfutils-libelf-devel ncurses-devel newt-devel numactl-devel pciutils-devel python-devel zlib-devel  bc bison java-devel python-docutils
```

执行完```rpmbuild -bp --target=$(uname -m) kernel.spec ```命令后，查看/usr/src/目录，可以看到有kernel目录了，linux/init.h头文件则位于``` /usr/src/kernels/3.10.0-1062.9.1.el7.x86_64/include```目录下。

这时，执行```gcc -I /usr/src/kernels/3.10.0-1062.9.1.el7.x86_64/include hello.c```不会报linux/init.h缺失了。

但报了另一个错。

> asm/linkage.h: No such file or directory

真是坎坷啊，看来Linux模块编译不能使用简单的gcc命令了。上网搜索，只能上make来编译了。

## make编译

首先将你的文件移到一个目录，这里假设为你的Home目录下的test目录(~/test)，因为等会编译会产生多个文件，避免跟前期目录混在一起。

把hello.c文件移入test目录，在test目录下新建一个Makefile文件，输入如下内容。
```makefile
obj-m += hello.o # 这里的hello.o名字对应你的hello.c文件

all:
        make -C /lib/modules/`uname -r`/build M=`pwd` modules
clean:
        make -C /lib/modules/`uname -r`/build M=`pwd` clean
```

保存退出，执行make编译。

编译生成多个文件，包括hello.ko。

## 加载模块

使用命令
```sh
# 装载模块
insmod hello.ko
# 卸载模块
rmmod hello
```
就会提示Hello, world了，如果没显示内容，可以执行```tail /var/log/messages```命令查看。没在屏幕上显示的原因是日志级别不够。

你也可以修改hello.c文件，将KERN_ALTER改为KERN_EMERG，重新编译装入模块，就可以在屏幕显示Hello, world了。