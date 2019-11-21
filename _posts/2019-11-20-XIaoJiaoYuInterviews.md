---
layout: post
title: Java-广州晓教育面试
date: 2019-11-20
tags: 面试
---
# 广州晓教育Java面试

昨天上午去到了广州晓教育面试Java开发，公司位于广州太古汇。到楼下后直接上楼找面试接待即可，首先会被要求做笔试。

## 笔试题

+ 实现一个去重的集合类，不能使用已有的Collection或者Map
+ 介绍过滤器Filter和拦截器Interceptor，画一下SpringMVC执行流程
+ 写一个你认为最有的单例模式
+ 扣库存问题，a. 指出下面代码产生脏数据的原因， b. 思考方案解决这个问题。
```java
public void discount(int goodsId) {
    if (checkCount(goodsId)) { // 判断是否有库存
        ... // 业务逻辑处理
        dal.exec("update goods set count = count-1 where goods_id=?", goodsId); // 更新商品库存
    }
}
```
+ 排课表有3个字段：id, start_time, end_time分别代表记录id、课程开始时间、课程结束时间，用sql找出课程冲突的记录

| id | start_time | end_time |
| -- | -- | -- |
| 1 | 08: 00 |  09:00 |
| 2 | 08:30 | 09:30 |

+ 继续上面的课程冲突问题，使用代码找出List\<A\> list中不冲突的对象（返回不冲突的对象集合），A定义入下：
```java
public Class A {
    String startTime;
    String endTime;
}
```
+ 组织架构表的设计和查询（树状结构的数据表设计和查询），找出pid=1的所有子节点（要求：不能使用递归）
```
        0
       / \
pid->  1  3
      / \
      2  4
        \
        5
```

## 一面
笔试完毕，一面开始。面试官开始看笔试题。就着你写的非最优的方案提出意见。

+ 笔试第一题怎么使用遍历，有其他方案吗？（使用map）
+ 笔试扣库存问题，除了synchronized还有其他吗？（加锁，乐观锁）
+ 你项目中使用了redis，如果我有一堆大的热点数据，要注意什么问题？
+ （他给你一道题，建表的ddl）这个建表语句有什么可以优化的吗？
+ InnoDB现在drop掉主键索引，然后在unique索引上建立主键，需要注意什么问题？


还有其他问题，记不太清了。总体难度一般。

## 二面
二面是部门大佬面试，看起来挺年轻的。

+ 内部系统间的调用，设计的时候需要考虑什么异常情况？
+ 看过哪些源码，这些源码用到了哪些设计模式？

其他的记不太清了，最后一题是算法题。
+ 做一道算法题吧，复原IP地址？（leetcode第93题 medium难度，链接：[复原IP地址](https://leetcode-cn.com/problems/restore-ip-addresses/)）