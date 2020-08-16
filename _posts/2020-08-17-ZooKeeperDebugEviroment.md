---
layout: post
title: Zookeeper源码调试环境踩坑记录
date: 2020-08-17
tags: ZooKeeper
---

# Zookeeper源码调试环境踩坑记录

## 1. 下载源码

本文基于zookeeper源码3.6.1版本。

zookeeper的源码在github上，国内下载极慢，还好国内代码托管平台gitee克隆了github上著名的开源项目，从这里下载快多了

![quicker_68a198a0-dda6-4866-8017-1dfe3a9a1faf.png](https://i.loli.net/2020/08/16/dPhZvFA1XHecOJ6.png)

## 2. maven下载jar包

项目clone到idea后，maven开始自动下载依赖的jar包。没想到速度竟然可以这么慢，索性直接改了maven的settings.xml配置，换成阿里云的maven仓库，果然速度蹭蹭蹭一下子下载完毕。

## 3. 找入口类

zookeeper是一个java项目，启动时有启动脚本，执行命令./zkServer conf/zoo.cfg即可。那么我们可以从这个脚本中找到最终执行的java启动类，为QuarumPeerMain这个主类。另外一种方法可以在zookeeper启动后，执行jps -l则可以看到主类了。

## 4. 启动报错

知道主类后，我们在idea中配置zookeeper的启动参数。zookeeper启动时需要一个配置文件，我们可以从源码的根目录下的conf文件中zoo_sample.cfg复制一份改名为zoo.cfg。然后启动main方法。

发现报如下错误：
```
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Exception in thread "main" java.lang.NoClassDefFoundError: com/codahale/metrics/Reservoir
	at org.apache.zookeeper.metrics.impl.DefaultMetricsProvider$DefaultMetricsContext.lambda$getSummary$2(DefaultMetricsProvider.java:126)
	at java.base/java.util.concurrent.ConcurrentHashMap.computeIfAbsent(ConcurrentHashMap.java:1705)
	at org.apache.zookeeper.metrics.impl.DefaultMetricsProvider$DefaultMetricsContext.getSummary(DefaultMetricsProvider.java:122)
	at org.apache.zookeeper.server.ServerMetrics.<init>(ServerMetrics.java:74)
	at org.apache.zookeeper.server.ServerMetrics.<clinit>(ServerMetrics.java:44)
	at org.apache.zookeeper.server.ZooKeeperServerMain.runFromConfig(ZooKeeperServerMain.java:132)
	at org.apache.zookeeper.server.ZooKeeperServerMain.initializeAndRun(ZooKeeperServerMain.java:112)
	at org.apache.zookeeper.server.ZooKeeperServerMain.main(ZooKeeperServerMain.java:67)
	at org.apache.zookeeper.server.quorum.QuorumPeerMain.initializeAndRun(QuorumPeerMain.java:140)
	at org.apache.zookeeper.server.quorum.QuorumPeerMain.main(QuorumPeerMain.java:90)
Caused by: java.lang.ClassNotFoundException: com.codahale.metrics.Reservoir
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:581)
	at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:178)
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:521)
	... 10 more

Process finished with exit code 1

```

冒失缺失了一个类，导入对应的maven坐标还是无法解决。最后找到一种方法，将该main方法放到test类下面即可。如QuarumPeerMainTest。将QuarumPeerMain的main方法复制到QuarumPeerMainTest类中。再次启动即可。

![quicker_b2faecd6-0335-47bb-a712-dcf813428e50.png](https://i.loli.net/2020/08/17/u9j63kpeXLz1Yvg.png)

以上，忙了大半天。