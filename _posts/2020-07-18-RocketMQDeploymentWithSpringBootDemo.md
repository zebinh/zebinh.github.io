---
layout: post
title: docker安装部署rocketmq和SpringBoot小实例
date: 2020-07-18
tags: 工具
---

# docker安装部署rocketmq和SpringBoot小实例

![quicker_4a8515b7-48f3-4b2b-bce1-f442135ab7c7.png](https://i.loli.net/2020/07/18/sRgUlu4DwYAOckQ.png)

rocketmq模型如上图所示，分为如下几个部分：
+ NameServer：主要用作注册中心，用于管理Topic信息和路由信息的管理
+ Broker：负责存储、消息tag过滤和转发。需将自身信息上报给注册中心NameServer
+ Producer：生产者
+ Consumer：消费者

由上各部分角色的功能可知，我们需要先安装启动NameServer，再启动Broker即可搭建完RocketMQ

## 1. 部署NameServer

首先下载镜像：
```sh
docker pull rocketmqinc/rocketmq:4.4.0
```

启动NameServer，暴露9876端口
```sh
docker run --name rmqnamesrv -d -p 9876:9876 rocketmqinc/rocketmq:4.4.0 sh mqnamesrv
```

启动完成后，可以curl 9876端口测试服务是否启动成功

![quicker_9e183af9-2317-49a7-8bc8-70505be9aa9f.png](https://i.loli.net/2020/07/18/ktyWG4up3Mw5eOb.png)

## 2. 部署Broker

RocketMQ是Java编写的程序，Broker和NameServer都在上面的镜像中，只是启动命令不同而已。

启动Broker
```sh
docker run --name rmqbroker -d -p 10911:10911 -p 10909:10909  --link rmqnamesrv:namesrv -e "NAMESRV_ADDR=namesrv:9876" rocketmqinc/rocketmq:4.4.0 sh mqbroker
```

--link 将NameServer容器起个别名，Broker中需要配置一个NAMESRV_ADDR参数指向NameServer地址。

同上，这里也可以使用curl localhost:10911验证下服务器是否启动

## 3. 部署RocketMQ可视化界面控制台

这一个步骤不做也可以通过Java等客户端访问到RocketMQ了，不过有可视化界面便于观察RocketMQ数据。不需要的可以跳过这一步

下载镜像：
```sh
docker pull pangliang/rocketmq-console-ng
```

启动容器：
```sh
docker run --name rmqconsole -d -p 8080:8080 --link rmqnamesrv:namesrv -e "JAVA_OPTS=-Drocketmq.namesrv.addr=namesrv:9876"  pangliang/rocketmq-console-ng
```

自此，也可以使用curl命令测试控制台界面是否成功启动。curl localhost:8080，如下表示启动成功。
![quicker_c6e5c0c7-4f8d-403f-8a19-3bd517c4b54c.png](https://i.loli.net/2020/07/18/ngdqG8TOrPXhZik.png)

宿主机也可以登录访问控制台界面。
![quicker_35196b39-6877-456c-a93b-42d3f71cd677.png](https://i.loli.net/2020/07/18/K2CMq8561bNsOof.png)

## 4. SpringBoot整合RocketMQ小实例

maven中先导入apache官方提供的starter
```xml
<!-- https://mvnrepository.com/artifact/org.apache.rocketmq/rocketmq-spring-boot-starter -->
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>
```

application.yml配置一个name-server地址，具体值看你的机器。

![quicker_214effe4-c2f1-48db-838d-4915c303ef38.png](https://i.loli.net/2020/07/18/DgKW2x7ztpjniIQ.png)

这里也可以通过accessKey和secureKey登录连接。默认配置在RocketMQ的配置文件中，默认值是：
```
accessKey: RocketMQ
secureKey: 12345678
```

生产者发送消息：
```java
@RestController
public class RocketController {

    @Autowired
    private RocketMQTemplate rocketMQTemplate;

    // 发送给Broker，默认会自动创建topic，topic和tag用冒号分隔
    @GetMapping("/rocket/send")
    public String rocketSend() {
        LocalDateTime currentTime = LocalDateTime.now();
        rocketMQTemplate.convertAndSend("rocket-topic-1", currentTime.toString());
        return currentTime.toString();
    }

    // 延时消息，RocketMQ支持这几个级别的延时消息，不能自定义时长
    // 1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
    @GetMapping("/rocket/delayMsg/send")
    public String rocketDelayMsgSend() {
        LocalDateTime currentTime = LocalDateTime.now();
        rocketMQTemplate.syncSend("rocket-topic-1:tag-2", MessageBuilder.withPayload(currentTime.toString()).build(), 2000, 3);
        return currentTime.toString();
    }
}
```

消费者：
```java
@Component
@Slf4j
public class RokcetServiceListener {

    @Service
    @RocketMQMessageListener(consumerGroup = "consumer-group-1", topic = "rocket-topic-1")
    public class Consumer1 implements RocketMQListener<String> {

        @Override
        public void onMessage(String s) {
            log.info("consumer1 rocket收到消息：{}", s);
        }
    }

    // RocketMQ支持两种消费方式，集器消费和广播消费
    @Service
    @RocketMQMessageListener(consumerGroup = "consumer-group-2", topic = "rocket-topic-1",
            selectorExpression = "tag2", messageModel = MessageModel.BROADCASTING)
    public class Consumer2 implements RocketMQListener<String> {
        @Override
        public void onMessage(String s) {
            log.info("consumer2 rocket收到消息：{}", s);
        }
    }
}

```