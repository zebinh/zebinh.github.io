---
layout: post
title: 优雅地使用SpringBoot注解写代码 SpringBoot常用注解
date: 2020-07-08
tags: Spring
---

# 优雅的使用SpringBoot注解写代码 SpringBoot常用注解

本篇文章想写一下SpringBoot注解的优雅使用方法。我们所有的工程都离不开配置，那就从配置涉及到的注解开始写起吧。

假设我们的项目是一个数据表结构的迁移项目，用到了两个数据源，一个MySQL的和一个PostgreSQL的。我们项目的目的就是读取MySQL的表结构，生成PostgreSQL的ddl建表语句，生成表结构。

那么第一步，我们就需要配置数据源属性。

## 1. 配置文件
application.yml配置文件

```yml
jdbc：
    mysql:
        enabled: true
        driver-class: com.mysql.jdbc.Driver
        url: this is a mysql url
        username: mysqlUsername
        password: mysqlPassword
    postgresql:
        enabled: flase
        driver-class: org.postgresql.Driver
        url: this is a postgresql url
        username: postgresqlUsername
        password: this is a postgresql password
```

## 2. @ConfigurationProperties将配置文件映射为Java Bean键值对类

写好配置文件后，Java要使用配置文件，肯定要先将配置文件封装成Java Bean。使用@ConfigurationProperties即可。如下：

![quicker_a4f9f571-9299-433f-a4a1-961709d013ae.png](https://i.loli.net/2020/07/08/W9b8IuRxa3tGm72.png)

## 3. 将Properties键值对类引入@Configuration配置类

![quicker_e1a8ae4a-af5f-4682-be57-4727545920b4.png](https://i.loli.net/2020/07/08/TusDjPhfZ5CxXE6.png)

这里的@Configuration配置类，肯定需要引入Properties键值对类。但是MysqlProperties类上的@ConfigurationProperties注解是不会往容器中注入自身bean的，所以需要@EnableConfigurationProperties启用。

接着就是enabled配置的妙用了，配置@ConditionalOnProperty，当指定的键enabled为指定的值true时，该配置类才生效。

## 4. 其他注解

+ @Bean：该注解往容器中注入bean，当Bean不是我们项目中的源码时，无法在其类上加@Component注解使其被容器托管，这是可以使用@Bean注解。
+ @Import：快速向容器中导入一个类，和@Bean类似都是导入Bean的作用，然而@Import的bean id是全类名，因此一般不会用id来获取bean。@Import配合实现了ImportSelector接口的类，可以有动态的选择的向容器中导入指定Bean。
+ @Profile：该注解指定了一个环境。SpringBoot中application.yml可以配置多个环境，如application-dev.yml, application-test.yml。SpringBoot应用启动时添加-Dspring.profiles.active=test指定环境，同时也会作用到@Profile注解上。