---
layout: post
title: 一个操作系统的实现-于渊
date: 2019-12-30
tags: 读书
---

# 一个操作系统的实现-于渊

操作系统是处于硬件和软件之间的，所以开发操作系统其实就是按硬件的要求来实现自己的功能。即微软是按照Intel手册来做开发的。
+ 附Intel开发者手册官网：https://software.intel.com/en-us/articles/intel-sdm
+ CPU手册：https://www.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.html


## 马上动手写一个最小的操作系统

计算机启动时先执行主板的BIOS程序进行自检，之后执行主引导扇区（主引导设备的第0个扇区512字节）上的代码，扇区最后两个字节以0x55aa结束。根据这个原理，就可以写一段比操作系统还早运行的代码，即我们说的最小的操作系统。实验使用bochs虚拟机来做，先编写我们的最小“操作系统”(这个操作系统的功能是显示Hello OS world!)。然后汇编后将二进制代码写入bochs虚拟的软盘的第0个扇区，将软盘设置为bochs虚拟机的主引导设备。我们的操作系统就能显示出来了。

BIOS加载主引导扇区代码后，跳转到0x7C00处执行，为什么是0x7C00呢，因为以前的操作系统内存只有32k，即最大内存地址为0x7FFFF，故将引导扇区的代码放到内存尾部，再留512字节当引导扇区的数据存储空间，引导扇区代码执行完成后，内存就不需要保留这段代码了，可以回收使用。

下图为最小的操作系统实验界面：
![UTOOLS1577455891933.png](https://i.loli.net/2019/12/27/WVzTrvdwSRluAjP.png)

## 保护模式

保护模式是相对实模式来说的，上面的最小的操作系统，内存寻址是使用[段地址：偏移地址]来表示物理地址。这种方式由很大的问题，就是应用程序可以随意访问内存，极其不安全。因此，在intel 80286时代，就实现了保护模式，保护模式下的寻址范围更大，内存访问也更安全。保护模式下的寻址变为了[选择子：偏移地址]，选择子指向了GDT全局描写符表，选择子中最低两位表明了特权级。GDT是一组Descriptor描述符，描述的是内存基址等信息，相当于实模式下的段地址。

实模式跳到保护模式，在于如下将cr0的第0位置为1，cr0第0位为1则表明处于保护模式下，接着jmp到SelectorCode32:0处，此时SelectorCode32要看成是选择子而不是段地址了。需要以选择子的方式解析SelectorCode32。

![20191228000632.png](https://i.loli.net/2019/12/28/KiMnscUgWT5pOP1.png)

下图为实模式跳转到保护模式实验界面(中部右边有个红色的P)：
![20191227221605.png](https://i.loli.net/2019/12/27/kfO7HnIwhSyrNYb.png)

## LDT

LDT和GDT差不多，都是描述符表，都存放描述符。GDT中存放的描述符可以是GDT类型的，直接计算出内存地址，也可以是LDT类型的，计算出LDT表的位置。LDT表中又可放描述符，指向特地段。

## 特权级转移

保护模式中，不同的代码段有不同的权限等级，通过CPL，RPL，DPL和一致代码段实现，这些标记在描述符中体现。为例解决这个问题，可以使用门描述符或TSS。

## 内存分页

内存分页是虚拟内存实现的基础，现代计算机基本都使用分页模式管理内存而不是分段模式。分段和分页的不同在于，逻辑地址通过分段机制转为线性地址，线性地址通过分页机制转为物理地址。如果分页机制没开启，那么逻辑地址通过分段机制转为线性地址，此地址就是物理地址。

一般内存分页采用多级分页，以缩小分页表的大小节省内存，这里讲二级分页。第一级表示页目录PDE，其指定了页表的基址，第二级表示页表PTE，其指定了页基址。内存分页原理通过CR3(Controll Register)控制寄存器指定页目录基址。PDE, PTE中都包含了权限信息。

内存分页的运作方式为，以mov ax, [0x1234h]为例，CPU先读取CR3的中的页目录基址得到页目录，然后根据0x1234前0~9位获得页表偏移，根据偏移获得页表项PDE。PDE中包含了页基址，加上0x1234中前10~20位表示页偏移，获得物理页的基址。物理页基址加上0x1234中最后12位，获得物理地址。

## 虚拟内存的实现
我们平时写汇编代码时，其中操作了计算机内存如0x1234，但我们发现再复制一份相同的代码，运行时发现同样操作0x1234的内存却不会发生数据篡改的现象，这就是虚拟内存的功劳。因为内存地址相同，但页目录不同，所以访问的是不同的物理地址。  

如下，调用同样一段代码（即同一个地址），但是切换了页目录，其物理地址就是不一样了的。
![20191229110515.png](https://i.loli.net/2019/12/29/yYGLMS3RdIHik2F.png)

## 中断

保护模式下的中断和实模式下完全不一样。保护模式下有中断向量表IDT，和GDT类似，也是存放描述符。IDT中的描述符有三种类型：中断门描述符，陷阱门描述符，任务门描述符。门描述符有”开门“的意思，意为对要进行的处理需特权校验，符合特定的权限后才能操作。

## 让操作系统走进保护模式

直观想法，开机执行主引导扇区中的代码，我们可以在主引导扇区中加载启动我们的操纵系统不就可以了吗。别忘了主引导扇区只有512个字节，用这512个字节写一段加载一个操作系统的代码，恐怕是不够的。因此我们需要一个中间模块，名为Loader，主引导扇区加载Loader，Loader中再做加载操作系统的事情，这样就不会有512字节的限制了。

## 进程

进程调度时需要切换进程，而进程运行的状态有：寄存器和内存。现代操作系统都是使用的虚拟内存，各进程间独立，故不需要保存内存状态，所以进程切换时只需要保存寄存器的状态。有pushad, pusha等批量保存寄存器的指令。

## 汇编分类

机器指令可以分为复杂指令集CISC和精简指令集合RISC，这是两种完全不同的机器指令。设计生产这两种指令集芯片的代表公司有Intel(CISC)和ARM(RISC)。随着科技的发展Intel的CPU越来越强，因此有8086汇编(16位指令)，x86汇编(32位)，x64汇编(64位)。汇编语言分为两种，nasm和masn，他们的区别类似高级语言有C语言，Java语言一样。

## 输入输出系统

使用中断进行响应

## 进程间通信IPC

微内核和宏内核，unix属于微内核设计，而linux属于宏内核。微内核理念是使得内核尽可能的少做事情，而专注于进程的调度，其他如IO和内存管理都由系统进程处理，进程间使用消息通信。而宏内核则把IO等包括在内核中，避免了内核态的频繁切换。

## 文件系统

硬盘驱动

## 内存管理

fork()函数