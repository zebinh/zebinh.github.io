---
layout: post
title: 算法-美-塞奇威克-人民邮电出版社
date: 2020-04-21
tags: 读书
---

# 算法-美-塞奇威克-人民邮电出版社

## 基础

本章介绍的是学习算法和数据结构所需要的基本工具。本书的算法是使用Java语言实现的，因此这一章讲了一些Java语言的基础。

我们关注的大多数算法都需要适当的组织数据，这就产生了数据结构，数据结构是算法的副产品或是结果。

## 排序

leetcode排序题目：https://leetcode-cn.com/problems/sort-an-array/submissions/

排序很重要。在计算时代早期，大家普遍认为30%的计算周期用在了排序上，如果今天这个比例降低了，可能的原因是如今的排序算法更加高效，而不是排序的重要性降低了。

排序的一种算法 -- 快速排序，被誉为20世纪科学和工程领域的十大算法之一。

大多数情况下，排序算法只需要两个操作，比较和交换。

+ 选择排序：首先，找到数组中最小的那个元素，将它和数组的第一个元素进行交换。依此类推。我们用一个表格表示它的循环，可以看到，它需要比较n*n/2次，交换n次(忽然发现表格在计算算法时间复杂度方面很有用)
+ 插入排序：将每一张牌插入到其他已有序的牌中。算法中经常有很多i，j的临界点，可以在循环中写主要的流程代码，在循环外做临界处理。类似卫语句。同时注意限制下标值不越界。
+ 希尔排序：插入排序的改进版，分组插入排序
+ 归并排序：基本点是将两个有序的数组归并成一个有序的数组

**重点：快速排序**：它可能是应用最广泛的排序算法了。
+ 快速排序：基本点是进行分区，找到中枢点

LeetCode中排序时间对比：
+ ![GRVjgg.png](https://s1.ax1x.com/2020/04/08/GRVjgg.png)
+ ![G7YLDS.png](https://s1.ax1x.com/2020/04/10/G7YLDS.png)
+ ![GLbuy4.png](https://s1.ax1x.com/2020/04/12/GLbuy4.png)
+ ![JpfVmQ.png](https://s1.ax1x.com/2020/04/14/JpfVmQ.png)
+ ![JE9Ca8.png](https://s1.ax1x.com/2020/04/16/JE9Ca8.png)

十大经典排序算法：https://www.runoob.com/w3cnote/quick-sort-2.html

刚好是红宝书中算法的顺序。

堆排序：堆的结构应该需要满足，删除最大元素和插入元素两个方法，也叫优先队列。堆化的过程中涉及上浮和下沉。堆排序的几个步骤：
+ 使用数组维护堆结构，下标为0的不存放元素按，k节点的父结点为k/2，子节点为2k和2k+1
+ 堆化，每次插入元素节点到数组尾部，堆的规模+1，并使节点上浮。
+ 排序，每次取根节点放到数组尾部，堆的规模-1，并使根节点下沉。



LeetCode中排序时间对比：
+ ![Je3jTx.png](https://s1.ax1x.com/2020/04/18/Je3jTx.png)
+ ![Je8qC8.png](https://s1.ax1x.com/2020/04/18/Je8qC8.png)

## 查找

leetcode查找算法题目：https://leetcode-cn.com/problems/binary-search/

查找一般会在有序的数据结构中进行才会比较高效，如有序数组，hash表，二叉查找树，红黑树。以下是leetcode中各排序的时间：
+ ![JerISA.png](https://s1.ax1x.com/2020/04/18/JerISA.png)

二叉查找树在最坏的情况可能存在线性关系，因此有平衡查找树。
+ 2-3查找树：由2-节点和3-节点构成，3-节点比二叉查树节点多存储一个键，2个键3条链接，2-3查找树当变为4-节点时，需要做变换，即树的生长。
+ 红黑树：理解了2-3查找树的话，红黑树就比较好理解了，红黑树其实就是2-3树的变种表示。红黑树有几个重要的函数，左旋、右旋和反转颜色。

红黑树和堆有点类似：
+ 堆中插入元素时，需要立即上浮，红黑树插入元素时，需要旋转和变色。

## 图

4种重要的图模型：无向图、有向图、加权图、加权有向图。

连通图通俗的讲，就是提起图中的任何一个顶点，都可以将图整个提起。如果图被分为两部分，则不是连通图。

树是一种连通图。树和图很像，API也类似。树中最重要的有left和right属性，获得左右节点，图中有adj方法，获得与该节点连接的所有结点。

+ (图的)生成树：它是一棵含有图的所有顶点的无环连通子图。
+ 最小生成树：加权无向图的一棵权值之和最小的生成树。

最小生成树算法应用非常广泛。如电路和航空领域。

## 字符串

我们通过交流成串的字符进行沟通，所以无数的重要而熟悉的应用程序软件都是基于字符串处理的，除了通用的排序、查找算法，字符串有特有更高效的排序和查找算法。本章考察一些字符串的经典算法。

字符串排序：
+ LSD低位优先的字符串排序算法，适合长度相等的字符串
+ MSD高位优先的字符串排序算法，这是通用的字符串排序算法

字符串查找：
+ 单词查找树

子字符串查找([leetcode](https://leetcode-cn.com/problems/implement-strstr/))
+ 暴力解法：![JK7r8J.png](https://s1.ax1x.com/2020/04/19/JK7r8J.png)
+ KMP算法：![J3KsyT.png](https://s1.ax1x.com/2020/04/21/J3KsyT.png)

正则表达式：非确定有限状态自动机NFA

+ 数据压缩：霍夫曼编码

## 背景

B树