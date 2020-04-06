---
layout: post
title: RabbitMQ实战指南-朱忠华-电子工业出版社
date: 2020-04-06
tags: 读书
---

# RabbitMQ实战指南-朱忠华-电子工业出版社

## RabbitMQ简介

消息中间件的作用：
+ 解耦
+ 冗余（存储）
+ 拓展性
+ 削峰
+ 可恢复性：处理消息的机器宕机之后，消息中间件可以存储消息
+ 顺序保证
+ 缓冲
+ 异步通信：消息中间件可以延迟消费

RabbitMQ具体特点：
+ 可靠性：消息确认发布、传输确认，持久化
+ 灵活的路由
+ 拓展性：可以拓展为集群
+ 高可用：可以设置队列镜像，单一RabbitMQ挂机后，有镜像提供服务
+ 多种协议
+ 多语言客户端
+ 管理界面
+ 插件机制

## RabbitMQ入门

![GurFP0.png](https://s1.ax1x.com/2020/03/30/GurFP0.png)

RabbitMQ的模型架构如图，生产者```Publisher```通过TCP连接```Connnection```与RabbitMQ服务器```Broker```连接，每个```Connection```中有多个```Channel```。```Broker```服务器中有多个```Virtual Host```，每个```Virtual Host```各自独立。```Publisher```发布消息Message到```Exchange```交换器，Message包含label(路由键)+payload(消息体)，```Binding```上带有```绑定键```，交换器根据自身性质和路由键和绑定键决定将消息投递到哪里。最终消息被保存在```Queue```队列中，```Consumer```监听队列获得消息。

注意：
+ 生产者将消息传到交换器后，交换器根据自身交换器类型和路由键、绑定键决定消息投递的方向。
+ 只有队列能存储消息，消息到达队列时已没有label了，只剩下消息体
+ 当多个Consumer连接到同一个队列时，队列上的消息将轮询发给每个Consumer而不是每个消息复制多份发给每个Consumer，即每个消息只能被成功消费一次

### AMQP协议

![Gu27y6.png](https://s1.ax1x.com/2020/03/30/Gu27y6.png)

如上图，AMQP协议每次发送数据时的指令。

## 客户端开发向导

本章讲了RabbitMQ Java客户端的开发，只摘录我的部分理解在此。

创建RabbitMQ各个组件时可以有不同的参数。
+ Exchange交换器：type类型、durable是否持久化(持久化的Exchange重启时不会消失)，autoDelete自动删除(当Exchange所绑定的queue都删除后，自动删除Exchange，默认不自动删除)，internal是否内置(若是，客户端程序无法发信息到此交换器，只有交换器能发到此交换器)
+ queue队列：durable是否持久化、exclusive排他(同Connection可见，其他Conntection不能创建同名队列，Connection断掉自动删除)、autoDelete自动删除(有消费者订阅过此队列，之后所有消费者取消订阅，则删除，生产者创建队列后无客户端连接是不会删除的)

RabbitMQ的消息存储在队列中，交换器的使用并不真正耗费服务器的性能，而队列会。衡量RabbitMQ的QPS只需看队列即可。

+ 消息投递模式：发送消息处理指明交换器和路由键，还可以指定消息的一些属性，如投递模式为2时，消息可持久化。可以设置消息的优先级。
+ 消费消息：有两种模式，推模式Push和拉模式pull。消费消息时可以设置是否自动确认。当然，消息发送方也可以设置发送确认，RabbitMQ的exchange收到消息时可以回调发送方设置的监听。

消费端的确认和拒绝：消息订阅有推和拉两种模式，订阅时可以设置是否自动确认，自动确认的消息在RabbitMQ将消息发出时移除，非自动确认的消息需要客户端手动确认。也可以拒绝该消息，拒绝时传递一个requeue参数，可以设置是否将该消息重洗入队等待发送，或者将其移除。

## RabbitMQ进阶

本章讲的是RabbitMQ的高级功能。

+ 交换器的属性：
   + mandatory属性为true时，交换器如果找不到消息的路由，会将消息返回生产者，为false则丢弃
   + immediate参数为true时，当路由器发现队列中不存在任何消费者，则不投递到该队列。若所有队列都无消费者，则将消息返回生产者。RabbitMQ 3.0去掉了该属性。

备份交换器AE：
+ 当mandatory为true时，无法路由的消息会返回生产者，为false时，无法路由的消息会被丢弃。除了这两者，还可以为交换器设置备份交换器，当消息无法路由时，会发送到备份交换器，备份交换器和mandatory一起使用，该参数无效。

过期时间TTL
+ 过期时间可以设置在队列，也可以单独设置在消息。一旦消息过期，就会变成“死信”。对于设置队列的TTL，一旦消息过期则从队列中抹去，而对于单独设置TTL的消息，过期后不会马上抹去，而是在消息投递给消费者之前判断的。

死信交互器DLX(也称死信队列)：
+ 当消息变为死信之后，他能被发送到死信交换器。
+ TTL和DLX可以做延时队列

![GtcFsg.png](https://s1.ax1x.com/2020/04/03/GtcFsg.png)

持久化，RabbitMQ分为三部分：
+ 交换器持久化：交换器不持久化的话，重启后元数据丢失，但消息不会丢失，因为消息存在队列中
+ 队列持久化：队列不持久化的话，重启后队列会消失，消息也消失
+ 消息持久化：如果消息不持久化，那么重启时消息会丢失，队列持久化只能保证队列的元数据不丢失，不能保证消息不丢失。

消息的可靠性：
+ 生产者确认：默认情况下，生产者发送消息后不知道消息是否到达了RabbtMQ服务器。可以通过两种方式确认
    + 事务机制：如channel.txSelect将channel设置为事务模式，每次发送消息前都需要发送事务指令，严重影响性能
    + 发送方确认：类似tcp批量确认，每个消息都编号，RabbitMQ会确认一个序号表示这个序号之前的消息都已确认。相比上面的事务机制，这里是异步的，而事务是同步确认每一条消息的。
    + 当然，生产者确认只能保证消息传递到了服务器的交换器。如果消息没有匹配的队列，那么消息也会丢失，发送方可以配合mandatory参数或备份交换器解决没有匹配队列时的处理方式。
+ 消费端要点：默认队列中的消息会轮询分发到订阅同一队列的消费者，RabbitMQ并不管消费者是否确认了该消息，这会造成性能差的服务器处理过慢。可以使用channel.basicQos设定每个消费者最多持有未确认的消息数量，这种机制类似tcp的滑动窗口，这种机制对于拉模式的消息消费无效。
    + 消息顺序性：消息的顺序性是从存入队列是开始的，不能认为是从生产者发送时开始的，因为有网络抖动。
    + 消息传输保障：
        + 最多一次：生产者随意发送，消费者随意消费
        + 最少一次：生产者开启发送确认，生产者需要使用mandatory参数或备份交换器确保消息无法路由时能保存到队列。消息和队列都需要进行持久化。消费者需要手动确认消息。
        + 恰好一次：无法保证消息恰好一次被消费，因为网络原因，消费者消费之后发送确认消息时刚好网络断开，连接重新建立时消费者会重新消费到消息。RabbitMQ无法保证恰好一次，不仅RabbitMQ目前大多数主流的中间件都无法保证恰好一次。
        

## RabbitMQ管理

RabbitMQ的应用工具：
+ rabbitmqctl：命令行工具
+ rabbitmq_management插件：图形化插件

### 多租户与权限

+ 添加vhost：rabbitmqctl add_vhost myhost
+ 显示vhost：rabbitmqctl list_vhosts
+ 删除vhost：rabbitmqctl delete_host myhost
+ 权限配置到vhost：rabbitmqctl set_permissions [-p vhost] {user} {conf} {write} {read}，如授予root用户可访问虚拟主机myhost并具备所有权限
```rabbitmqctl set_permissions -p myhost root ".*" ".*" ".*"```
+ 清除权限：rabbitmqctl clear_permissions [-p vhost] {user}
+ 查看权限：rabbitmqctl list_user_permissions {user}

### 用户管理

+ 创建用户：rabbitmqctl add_user {username} {password}
+ 改密码：rabbitmqctl change_password {username} {password}
+ 清除密码：rabbitmqctl clear_password {username} {password}
+ 验证密码：rabbitmqctl authenticate_user {username} {password}
+ 删除用户：rabbitmqctl delete_user {username}
+ 查看用户：rabbitmqctl list_users
+ 设置角色：rabbitmqctl set_user_tags {username} {tags...}

用户分为五种角色：
+ none：无任何角色，新创建的用户默认为none角色
+ management：可以访问web管理页面
+ policymaker：包含management的所有权限，并可以管理策略和参数
+ monitoring：包含management的所有权限，并可以看到所有连接、信道和结点
+ administrator：管理员

### web端管理

+ 启动插件命令：rabbitmq-plugins enable rabbitmq_management
+ 查看插件使用情况：rabbitmq-plugins list 

### 应用与集群管理

+ 停止服务：rabbitmqctl stop，rabbitmqctl stop_app，rabbitmqctl shutdown
+ 重置：rabbitmqctl reset还原rabbitmq到最初状态

vhost、权限等都可以使用rabbitmqctl来创建，但交换器、队列和绑定关系却无法用rabbitmqctl来创建，只能通过web管理界面创建。然而，rabbitmq management还提供了命令行rabbitmqadmin来解决这个问题。如查看队列中的消息
```
rabbitmqadmin get queue=myqueue
```

rabbitmq management还提供了http api接口来方便调用。如创建一个队列，可以使用PUT方法调用/api/queues/vhost/name接口来实现。

## RabbitMQ配置

三种方式配置：
+ 环境变量：如RABBITMQ_NODENAME=rabbit@node2 rabbitmq-server -detached，环境变量也可配置在```RABBITMQ_HOME/etc/rabbitmq/rabbitmq-env.conf中```
+ 配置文件：配置文件位于/opt/rabbitmq/etc/rabbitmq/rabbitmq.config
+ 运行时参数和策略：可以通过rabbitmqctl或management插件提供的http api来设置，如rabbitmqctl set_parameter 

## RabbitMQ运维

本章主要将集群的搭建和运维

## 跨越集群的界限

RabbitMQ可以通过3种方式实现分布式部署：
+ 集群
+ Federation：该插件的目的是使RabbitMQ在不同的Broker结点之间进行消息通信而无需创建集群
+ Shovel：该插件能够可靠、持续地从一个Broker中的队列拉取数据并转发至另一个Broker中的交换器。

## RabbitMQ高阶

本章是将RabbitMQ的内部原理的，需要熟悉

### 流控

RabbitMQ可以对内存和磁盘使用量设置阈值，达到阈值后，生产者将阻塞。这是全局流控，面向所有连接的。当然RabbitMQ也可以面向单个Connection进行流控。

RabbitMQ使用一种基于信用证算法的流控机制来限制发送消息的速率。

### 镜像队列

生产者发送消息时，会将消息都发送到各个slave，除发送消息外的所有动作都只会向master发送，然后再由master将命令执行的结果广播给各个slave。

### 网络分区

在局域网环境下，网络设备出现故障时也会导致网络分区。当出现网络分区时，不同分区里的结点会认为不属于自身所在分区的结点都已经挂了。如果原集群中配置了镜像队列，每个网络分区中都会出现一个master结点。

## RabbitMQ拓展

消息追踪：Firehose

负载均衡：轮询、加权轮询、随机、加权随机、源地址哈希、最小连接数

使用HAProxy实现负载均衡

使用Keepalived实现高考可靠负载均衡

使用keepalived+LVS实现负载均衡