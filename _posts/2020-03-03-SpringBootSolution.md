---
layout: post
title:  Java工程搭建方案 - Spring Boot技术解决方案
date: 2020-03-06
tags: 工程
---

# Java工程搭建方案 - Spring Boot技术解决方案

现在的Java项目，基本可以说都是基于Spring Boot搭建的项目了。而一个完整的基本项目的构成中，有如下几个功能是很常见的：
+ 定时任务
+ 异步
+ 缓存
+ 消息队列
+ 权限控制

幸运的是，以上的功能Spring Boot都已经给我们准备了开箱即用的方法。

## Spring Boot定时任务

Spring Boot自带的注解@Scheduled加在方法上，并在应用启动类上加上@EnableScheduling即可。十分方便，如下所示。相比以前使用Quartz框架，不需要继承Quartz的Job接口并实现他的execute方法，不需要再去导入jar包。

![UTOOLS1583327096568.png](https://user-gold-cdn.xitu.io/2020/3/4/170a5a62d450f008?w=555&h=197&f=png&s=15008)

## Spring Boot异步

Spring Boot自带的注解@Async加在方法上，会将该方法变为异步方法，前提是需要在启动类上加上@EnableAsync注解。该方法还可以返回Future对象，获取异步线程的执行结果。@Async注解的属性中还可以指定线程池。这在系统应用中对于将接口实现异步化提高效率很有帮助。在此对比传统的线程池操作：
+ 以前需要new ThreadPoolExecutore定义线程池，现在可以通过@Bean添加线程池
+ 以前异步需要显式继承Runnable或Callable或继承Thread类，实现run方法，现在只需添加@Async注解
+ 以前线程需要提交到线程池，现在只需要在@Async("poolName")属性中写线程池名字

## Spring Boot缓存

Spring Boot中支持spring-cache进行声明式缓存，不像以前旧的方式@Autowired注入一个Jedis然后进行手动缓存操作，spring-cache只需要添加注解即可完成缓存动作。spring-cache底层缓存可以使用ConcurrentMap或者Redis，当然目前大多数缓存方案都是使用的Redis，同时Redis推荐的Java客户端是Redisson，因此spring-cache整合Redisson是较好的选择。我们来对比一下如果我们自己使用Map来做缓存和使用Spring Boot注解来做缓存的区别。
+ 执行一个方法前，我们会从一个Map(假设为accountCache)中获取值，如果获取不到则执行方法并将结果放入缓存，等同于@Cacheable("accountCache")注解，key就是参数(假设为userName)，value就是方法返回值
+ 清空缓存，@CacheEvict(value="accountCache", key="#account.getUserName()")
+ 更新缓存，@CachePut，执行方法并更新缓存

值得注意的是，spring-cache不支持缓存定时失效，需要自定义CacheManager即可。

## Spring Boot消息队列

Spring Boot使用了RabbitTemplate和@RabbitListener来发送和监听消息队列，比起单纯引用AMQP的jar包来开发真的是节省了很多代码和时间。基本用法是，向Spring容器中注入RabbitTemplate实例，并注入Queue队列，使用RabbitTemplate向指定的队列发送消息。监听者使用@RabbitListener监听指定的队列（该注解在方法上，也可以在类上，后者需配合@RabbitHandler注解使用），执行方法。

## 权限控制

权限控制一般使用Shiro，和Spring Boot独立，另开文章讲解。

## 综合

综上，Spring Boot使用了注解为我们开发提供了很多便捷，基本上都是注解在方法上对一个操作进行简化。善于利用Spring Boot提供的功能会事半功倍。