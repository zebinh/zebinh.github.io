---
layout: post
title: 十分钟完成的操作系统att版本
date: 2020-09-19
tags: 操作系统
---

# 十分钟完成的操作系统at&t版本-《一个操作系统的实现》

以前看了《一个操作系统的实现》这本书，使用了nasm汇编和bochs虚拟机来编写一个“操作系统”。做了清华大学ucore实验后，我感觉使用at&t汇编和qemu虚拟机来实现一个“操作系统”更加容易调试操作系统，因此改写了成at&t版本。

at&t汇编版本代码如下：
```at&t
.code16
.section .text
.global _start 
_start:
  movw %cs, %ax
  movw %ax, %ds
  movw %ax, %ss
  call DispStr
loop1:
  jmp loop1

DispStr:
  movw $msg, %ax
  movw %ax, %bp
  movw $16, %cx
  movw $0x1301, %ax
  movw $0x000c, %bx
  movb $0x00, %dl
  int $0x10
  ret
msg:
  .ascii "Hello, OS world!"
  .org 510
  .word 0xAA55
```

接下来使用as汇编、ld链接：
```bash
as -o boot.o boot.s
ld -Ttext=0x7c00 --oformat binary -o boot.bin boot.o
```

接下来制作一个512KB的虚拟硬盘，将上面生成的“操作系统”写入第一个扇区。
```bash
dd if=/dev/zero of=boot.img count=1000
dd if=boot.bin of=boot.img conv=notrunc
```

接着启动qemu虚拟机，如下，就是模拟插入boot.img硬盘，不需要配置文件，比bochs更方便。
```bash
qemu-system-x86_64 -hda boot.img -monitor stdio
```

成功，如下图：

![quicker_d7118466-811e-41f7-a0e9-d2409e098768.png](https://i.loli.net/2020/09/19/bX5aALNwiYrI2lx.png)

qemu不用配置文件真的很方便，加上启动参数-S -s就能使用gdb单步调试了。