---
layout: post
title: 从RocketMQ源码了解其系统设计
date: 2020-08-02
tags: 工具
---

# 从RocketMQ源码了解其系统设计

文章写作时间：2020年7月30日 07点50分

本篇文章RocketMQ代码基于最新的源码：rocketmq-all-4.7.1。

工作中经常用到RocketMQ，只知道使用却不知道他的原理，有时候排查问题都不知从何处下手。所以最近研究了一下RocketMQ的源码，了解其系统设计，使用起来也得心应手了。

读了这篇文章，你会了解到RockeMQ的架构和解决RocketMQ以下疑问。
+ RocketMQ消费者消费失败后为什么会重试？
+ RocketMQ延时队列的原理？
+ RocketMQ如何将一条消息同时发送给外部的A、B系统？
+ 内网环境如何避免其他同事的机器抢你的MQ消息？
+ RocketMQ消息进度如何保存？

![quicker_8f6133a3-cf05-4d59-a801-f9a50848bab7.png](https://i.loli.net/2020/07/29/TPumAUEBZcr956I.png)

以上是我画的RocketMQ的系统设计图。本来想用Excel画一个，嫌画起来麻烦就用apple pencil画一个，将就着看吧，主要看我接下来文字的讲解。

## 消息发送总流程

如果只是想了解RocketQM的消息发送和消费流程的话，阅读这部分的文字就够了。下面文字讲讲上面图的流程。

RocketMQ有四个概念模型，RocketMQ中间件启动时，首先启动NameServer，NameServer维护了Broker的信息，它接收Broker的心跳，Broker每隔30s会想NameServer发送心跳。NameServer并每10s扫描一下存活Broker，如果超过120s没有收到Broker的心跳，则认为该Broker宕机，将Broker的信息移除。

Broker消息代理服务器，他是保存和转发消息的服务器。Broker启动时，每隔30s向NameServer发送心跳。并且接收生产者和消费者的请求。消息存储在Broker中，Broker接收到消息后都将其写入commitlog目录中，并写入到指定的队列，队列在Broker中的组织形式是consumerqueue目录下的topic名字命名的目录，每个队列以0、1、2、3数字命名。写入到commitlog后会将该消息的offset转发到consumerqueue中，消费者才能从队列中拉取消息。

生产者启动时每隔30s向NameServer更新topic路由信息。生产者发送消息时，先在本地缓存中查找他要发送的topic在哪个Broker，如果找不到则请求NameServer查找。找到Topic路由和Topic下有几个队列后，生产者选择该topic下的一个队列，将消息发送到该队列。这里还涉及到故障规避算法，这里不讲解。

RocketMQ的消费者获取消息的方式有推模式和拉模式。推模式其实就是客户端封装的拉模式，本质上RocketMQ只有一种拉模式。消费者会根据自身的负载均衡算法，选择本消费者需要消费的队列。请求Broker获取该队列的信息，默认32条消息。将消息拉取下来后，会将消息提交到线程池进行消费，默认10个线程。如果消费失败，会将消息发回Broker失败重试机制，并推进内存中的进度。如果成功则直接推进进度，消费者每隔10s会将消费进度发送到Broker保存。

如果消费失败，Broker会将消息投递到延时队列中，所以RocketMQ的失败重试机制是基于延时队列的。RocketMQ只支持特定延时等级的延时队列，不支持自定义延时时间。每个延时队列，都有一个定时器每100ms查询该消息是否到时间了。是的话则将消息发送给对应的队列。

## RocketMQ模型概念

![quicker_49b7ec23-9bcb-4483-906c-9c7f8b825c38.png](https://i.loli.net/2020/07/30/MFLnhZePlRiGcX7.png)

如上图，RocketMQ主要涉及4个角色：
+ NameServer：命名服务器，更新和发现Broker服务
+ Broker：消息中转角色，负责存储、转发消息
+ Producer：生产者
+ Consumer：消费者

## RocketMQ路由中心NameServer

NameServer主要提供服务发现和注册功能，是一个无状态节点，可集群部署，集群间的机器不交流。

源码中，NameServer启动类为NamesrvStartup，启动时它每10s扫描一个存活的broker(非存活会移除)，并启动Netty处理网络请求

![quicker_bb057244-b60d-4b1f-a174-dd095c0e48d5.png](https://i.loli.net/2020/07/30/jVYPFpd5SJfn6aO.png)

#### NameServer路由注册和故障剔除

NameServer用了4个Map来维护Topic路由信息。

![quicker_d6aff96f-f69b-4d22-b7e9-1ace84725e12.png](https://i.loli.net/2020/07/30/cOsdfuYTo3tjNDU.png)

他们的运行时结构如下：

![quicker_431af6cd-779f-46d5-b621-0618c0b537bb.png](https://i.loli.net/2020/07/30/h4oAFLTRu5kewYy.png)
![quicker_0873bdbb-7fa3-4a0d-9a3c-da5faaaa1754.png](https://i.loli.net/2020/07/30/BzmnASscp95gYRx.png)

+ topicQueueTable：Topic队列表，记录每个Topic下有多少个队列，在哪个Broker上，消息消费是基于队列的。
+ brokerAddrTable：Broker地址表，记录Broker在哪个集群，Broker的主从地址。
+ clusterAddrTable：集群信息表，记录了集群下所有的Broker
+ brokerLiveTable：broker存活信息表，记录了broker上次心跳时间，超过120s表示broker宕机，移除该broker

#### Broker心跳

broker每10秒发送心跳。

![quicker_ba35d4a0-ed30-4fc9-9266-997edd96e081.png](https://i.loli.net/2020/07/30/pKWhdtFXfTEDqAy.png)


#### 路由发现

RockeMQ路由发现是非实时的，当Topic路由出现变化后，NameServer不会主动推送给客户端(生产者、消费者)，而是由客户端定时拉取主题最新的路由。

![quicker_df9e61ed-7a87-4461-b777-45a5cbef71df.png](https://i.loli.net/2020/07/30/mrgNQ6FaGIfV1uk.png)

## RocketMQ消息发送

RocketMQ支持3种消息发送方式：
+ 同步
+ 异步
+ 单向

主要了解的两个问题：
+ 消息队列如何进行负载？
+ 消息发送如何实现高可用？

### Producer启动

Producer启动时，会获取一个MQClientInstance客户端，用于和服务端交互。

![quicker_e58374f2-bb5c-4145-93af-d0bb67e6c21d.png](https://i.loli.net/2020/07/30/BrHJduhK9jTFpGE.png)

同一个JVM中只有一个MQClientInstance，不管有多少生产者或者消费者。路由信息由MQClientInstance维护，所以生产者和消费者使用的是同一份路由。

### 查找主题路由信息

本地缓存中没有Topic的路由，则实时向NameServer拉取。拉取不到，如果配置了自动创建Topic，则自动创建。

![quicker_e08b0ea3-c569-4106-9fd3-0b0116751ff5.png](https://i.loli.net/2020/07/30/Z7LoyhHwB9q82ps.png)

Topic在Broker上的存在形式是一个目录，队列MessageQueue才是存储的实体。消息的发送和接收都基于队列。

![quicker_baf76ca8-2a13-4372-8482-bd7fbe1633e0.png](https://i.loli.net/2020/07/30/k32zFV5sRn8GLuf.png)

### 选择消息队列

选择一个队列，向这个队列发送消息。默认的负载均衡方式就是每次队列序号+1

![quicker_d4ff8ba5-251d-4dfe-af56-d85dfa3e00bd.png](https://i.loli.net/2020/07/30/KTeZhyGANrfXszR.png)

如果开启了故障规避，发送给某个Broker失败，则会规避一定的时间。所以高可用的方式就是重试与规避。

![quicker_90719599-1617-4982-925a-e69fa0a935a1.png](https://i.loli.net/2020/07/30/p7oixGS6FwJeRKO.png)

## RocketMQ消息存储

RocketMQ消息存储主要有三个文件：
+ commitlog：所有消息发送过来都会落到这个文件中
+ consumerqueue：这个目录下是很多以topic命名的目录，每个topic下有多个队列目录，每个队列下有多个文件。消息落到commitlog后会异步转发到这个目录中。消息的消费以这个目录为主
+ index：提供了以tag快速查询消息的索引文件

consumerqueue和index中查到的只是消息的偏移量，查到偏移量之后再到commitlog中查整个消息的信息。类似mysql的B+树索引，根据普通索引找到的是主键id，再根据主键id回表查询记录。存储文件这么设计使得写入commitlog时是顺序写，提高了效率。

commitlog中每个文件1G大小，使用了系统调用mmap()内存映射文件技术。

![quicker_51d64699-a648-4453-8dd3-c7638e9a5693.png](https://i.loli.net/2020/07/30/XhxQKVLHSCGrYTp.png)

示例图

![quicker_eff18628-ab95-40f7-afb5-23ca26c09bc9.png](https://i.loli.net/2020/07/30/5tfAhCBKHD36Gkw.png)

消息id(msgId)的生成方式如下，所以可以通过msgId快速从commitlog中找到消息。

![quicker_1747134d-6da3-474b-a8d9-638e70446311.png](https://i.loli.net/2020/07/30/zJ1E43M9p5VgcGe.png)

### RocketMQ相关文件

consumerOffset.json存储了集群消费模式消息消费的进度，以消费组的形式存在，如果删除了这个文件，可以设置消费组默认从commitlog的头或者尾开始重新消费。

![quicker_b38248b6-c892-4586-bf35-2e8c330a8aa0.png](https://i.loli.net/2020/07/30/oXjVuq6JNZYOMl4.png)
![quicker_579f8ea9-5d18-4e5e-a3e3-0e43941c988a.png](https://i.loli.net/2020/07/30/nm7kAtXUij9MEZ3.png)

consumerqueue消息队列文件中存储的结构如下，最后的tagcode在消费者拉取消息时起到了过滤的作用。

![quicker_e7e3219d-750e-4fc3-baf3-60af35134207.png](https://i.loli.net/2020/07/30/j3sUQNE6wqdSoC1.png)

消息存到了consumerqueue队列中时，如果开启了长轮询，会实时将消息返回给消费者。

![quicker_a1e37488-1588-4896-b436-43fecb2f9f0a.png](https://i.loli.net/2020/07/30/wr72uRsB5aWlx6d.png)

### 过期文件定期删除

默认保留3天

![quicker_e337bdf3-532b-44a4-a822-7f1fd2c238f1.png](https://i.loli.net/2020/07/30/yCfc8lILaqSNiM7.png)

![quicker_f2b57e53-ce74-47fd-8b53-747e8248f266.png](https://i.loli.net/2020/07/30/zPRrZ1qVwp3SDjJ.png)

## RocketMQ消息消费

消息消费以组的模式开展，一个消费组内可以包含多个消费者，每一个消费组可订阅多个主题，消费组之间有集群模式与广播模式两种消费模式。集群模式，主题下的同一条消息只允许被其中一个消费者消费。广播模式，主题下的同一条消息将被集群内的所有消费者消费。消息服务器与消费者之间的消息传送也有两种方式：推模式、拉模式。所谓的拉模式，是消费端主动发起拉消息请求，而推模式是消息到达消息服务器后，推送给消息消费者。RocketMQ消息推模式的实现基于拉模式，在拉模式上包装一层，一个拉取任务完成后开始下个拉取任务。


### 消费者启动

消息消费者启动时会订阅指定的主题和重试主题。

![quicker_569e9842-b2a5-4a1d-9549-104baf88dcc8.png](https://i.loli.net/2020/07/31/vKbTAgta1JqkLpe.png)

由RebalanceImpl进行负载，就是将主题下对应的队列分配给不同的消费者拉取。

![quicker_4127f482-3ba8-4108-9159-c62aad214578.png](https://i.loli.net/2020/07/31/LR8y5WxJkUDKPSH.png)

查出所有队列，进行负载分配。有多种负载均衡算法。负载会定时执行，如果新增加了消费者则会将该消费者加入负载均衡中。如果消费者个数大于队列数，多出来的消费者将不参与消息消费。

![quicker_122a1518-7d73-4196-acb6-b34f7befc49a.png](https://i.loli.net/2020/07/31/XpO2TvnrezQ4kZt.png)

### 拉取消息

分配好负载后，每个消费者都分配到了指定的队列。开始拉取消息，并提交到线程池消费。每次默认拉取32条消息。为提高效率，消息拉取采用长轮询机制。

![quicker_884156c8-2cc0-423a-8d39-838ef4b508c0.png](https://i.loli.net/2020/07/31/vmf2ENahZdQH16b.png)

消费模式有并发消费和顺序消费。

![quicker_366dc75a-aa5e-4951-b534-fb222a3de197.png](https://i.loli.net/2020/07/31/KOt6Hnug4FUlkEB.png)

开始消费消息，执行业务逻辑。默认10个线程，最大20个线程。

![quicker_0b011995-68c1-4ce3-b6c8-8ddee508b45b.png](https://i.loli.net/2020/07/31/vceCguNUSrBkhm9.png)

消费失败，将消息发回Broker，失败重试。推进消费进度。

![quicker_e4c61ddf-2eb4-40fd-bb66-8dcaa610aa70.png](https://i.loli.net/2020/07/31/Drq97KbQEhH8Azw.png)

消费失败设置delayLevel

![quicker_0546cbea-2e6a-4f68-9a95-fdce4f728aed.png](https://i.loli.net/2020/07/31/I6uHdX1BUb4sRJz.png)

## 定时消息机制

![quicker_8fcd3156-c159-4081-a796-68ad90e0b116.png](https://i.loli.net/2020/07/31/DpCQHqw6OYj5y9Z.png)

每个定时消息队列有一个定时器，定时器的任务就是遍历定时队列，将时间到了的消息封装成指定的topic写回commitlog。间隔100ms后定时器重新扫描定时队列。

![aM6Hzt.png](https://s1.ax1x.com/2020/07/31/aM6Hzt.png)

定时消息图：

![quicker_342e4e3b-8525-4e35-8cc1-01fe99f0d5d3.png](https://i.loli.net/2020/07/31/xP3syKcow7YDGTq.png)

## 消息重试机制

定时消息时通过设置delayLevel声明，消息消费失败时，也设置了delayLevel，所以消息重试机制是基于定时消息的。每次delayLevel加一，如果失败超过16次，则不会重试，投递到死信队列。

![quicker_917e1090-5f2f-4aeb-8306-8fafef52c456.png](https://i.loli.net/2020/07/31/AFV6jBgwO7D1xi3.png)


## 服务端处理消息拉取

服务端会以tagcode先过滤一遍。

![quicker_28b34382-8ce5-4aec-bd2f-788514cae716.png](https://i.loli.net/2020/07/31/2sKJPLXa4xBijpI.png)