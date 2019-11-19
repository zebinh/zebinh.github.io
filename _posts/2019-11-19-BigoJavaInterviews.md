---
layout: post
title: 广州Bigo面试-Java
date: 2019-11-19
tags: 面试
---
# [Java]广州Bigo面试
本文作于2019年11月19日。

昨天投递简历到广州Bigo公司，接到邮件约定今天去面试。Bigo公司环境挺高大上的，无奈二面被刷了。在此记录下面试问题，继续努力。

## 一面基础
到公司签到后，hr下楼带上会议室面试。

一面是一个30岁左右的年轻人，第一各问题当时是自我介绍了。介绍完进入主题。

1. 为什么String是不可变的？（final？没那么简单，不懂了）
2. StringBuilder和StringBuffer区别？（线程安全？继续深究）
3. 我能定义一个java.lang.Object的类吗？（类加载机制）
4. Map了解吗？有哪些实现？（HashMap, ConcorrentHashMap, TreeMap应该还有其他，只说了这三个）
5. 上面我说到了ConcorrentHashMap，他问，分段锁是怎么实现的？JDK1.7和JDK8中ConcorrentHashMap实现的区别？
6. 线程池了解吗？（了解，我看过源码，说了核心线程数、最大线程数、队列和拒绝策略）
7. JVM OOM怎么排查？
8. gc的过程？
9. 线程不是有Idle时间吗，底层怎么做的？（懵逼）
10. 来个算法题？（剑指offer原题，easy级别，牛客网原题44题，链接：[反转单词顺序列](https://www.nowcoder.com/practice/3194a4f4cf814f63919d0790578d51f3?tpId=13&tqId=11197&tPage=3&rp=3&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)）


## 二面项目

二面是一个30多岁的年轻大叔，首先自我介绍。然后开问。

1. 挑一个简历上你熟悉的项目，画各架构图？（我画了Spring Cloud微服务那套，Nginx => Zuul => 微服务 => Mysql，外加Eureka、Feign这些）
2. 看你这个这个Nginx只有一台吗，单点故障怎么解决？（用keepalived做集群）
3. keepalived是怎么解决的？期间是所有的Nginx都有流量吗？（keepalived通过脚本检查健康状态，不是所有Nginx都有流量，只是主备）
4. 我想多台Nginx都有流量，怎么解决？（不懂）
5. Zuul的作用是什么？（网关路由）
6. Zuul还有其他作用吗？（提示了限流）
7. Feign是怎么进行客户端调用的？（说了集成Ribbon，Hystrix）
8. Ribbon有哪些负载均衡方式？怎么实现同一个客户端转发到同一个微服务？（后者提示了Ribbon自定义策略）
9. 你说Hystrix能熔断，它是怎么判断什么时候需要熔断的？
10. 项目中Eureka集群怎么互相同步数据？
11. 数据库、Redis、系统缓存间怎么保持数据一致？
12. Reidis使用大的value会有什么问题？
13. synchronzied和volatile作用？

以上，大部分回答都深究到底层，架不住啊，二面就回来等通知了。