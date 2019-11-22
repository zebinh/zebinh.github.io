---
layout: post
title: Spring Boot启动流程和自动配置原理
date: 2019-11-22
tags: Spring
---
# Spring Boot启动流程和自动配置原理

Java程序员应该都知道，每个Spring Boot都有一个启动类，Spring Boot的启动就是执行的该类的main方法。如下图，main方法中就是执行SpringApplication.run方法。

![20191122202613.png](https://i.loli.net/2019/11/22/kr1N6oegGLQBAOH.png)


## 启动流程

先总结一下Spring Boot启动流程。
+ SpringApplication.run中执行了两步操作，先封装了一个SpringApplication的实例，再执行该实例的重载run方法
+ SpringApplication封装实例时，读取了classpath下所有的```MTEA-INF/spring.factories``` xml配置文件的ApplicationContextInitializer（容器初始化器）还有ApplicaiontListener（侦听器），将这两者封装到SpringApplication实例中
+ 执行SpringApplication实例的run方法
+ run方法中默认初始化了Annotation配置的容器AnnotationConfigApplicationContext
+ 执行上面ApplicationContextInitializer的initial方法
+ 然后加载Bean到容器中


## Spring Boot自动化配置

我们知道，使用Maven坐标导入开发所需的jar包后，同时一些默认配置也会生效。那么Spring Boot又是怎么为这些jar包配置默认值的呢？ 答案就在Spring Boot的启动类上的注解@SpringBootApplication中。

![Spring Boot启动类](https://upload-images.jianshu.io/upload_images/8746907-7a7273c5c441ac0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

@SpringBootApplication主要由三个注解构成：```@SpringBootConfiguration```、```@EnableAutoConfiguration```、```@ComponentScan```

![@SpringBootApplication](https://upload-images.jianshu.io/upload_images/8746907-fd1dabdd2a4da33f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```@EnableAutoConfiguration```底层是由两个注解组成，```@AutoConfigurationPackage```、```@Import(AutoConfigurationImportSelector.class)```
![@EnableAutoConfiguration](https://upload-images.jianshu.io/upload_images/8746907-90c0499dc4ff4e5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```@Import(AutoConfigurationImportSelector.class)``` 自动配置的奥妙就在这里啦，这个类导入了很多自动配置类，debug一下可以发现，其读取的是classpath下的```META-INF/spring.factories```下的自动配置类
![自动配置类](https://upload-images.jianshu.io/upload_images/8746907-b01fca5cb050409f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

自动配置类如下：
![自动配置类](https://upload-images.jianshu.io/upload_images/8746907-1c0475ebcdb8f9b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 总结

Spring Boot通过主启动类上的@SpringBootApplication中的@EnableAutoConfiguration读取了类路径下的```META-INF/spring.factories```下EnableAutoConfiguration的配置类，但是这些配置类使用了@ConditionalOnClass，需满足一定的条件才会激活配置，这些配置类写入了默认的配置。
