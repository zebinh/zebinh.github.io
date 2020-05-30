---
layout: post
title: Spring揭秘-王福强-人民邮电出版
date: 2020-05-30
tags: 读书
---

# Spring揭秘-王福强-人民邮电出版

## 第3章 掌管大局的IoC Service Provider

IoC Service Provider两个职责：
+ 业务对象的构建管理
+ 业务对象间的依赖绑定

Spring的IoC Service Provider是BeanFactory，Spring的IoC容器不仅提供IoC Service Provider，还提供了其他的如线程管理、Aop支持等功能。

BeanFactory和ApplicationContext
+ BeanFactory：只提供基本的IoC Service Provider，默认采用延迟初始化策略
+ ApplicationContext：构建在BeanFactory之上，还提供了其他高级特性，默认初始化所有对象和依赖关系。

如果你需要自定义某一个类的生成规则，你可以使用FactoryBean，FactoryBean是一个生产Bean的工厂类接口。

### 战略性观望IoC容器“黑匣子”

Spring的IoC容器实现其两个职责时，流程可以划分为两个阶段：
+ 容器启动阶段：BeanDefinitionReader读取bean配置，解析成BeanDefinition，注册到BeanDefinitionRegistry，这样容器就启动完成了。
+ Bean实例化阶段

![YxOfX9.png](https://s1.ax1x.com/2020/05/24/YxOfX9.png)

每个阶段都加入了相应的容器拓展点(即钩子函数)，以便我们拓展逻辑。
+ 插手“容器的启动”：Spring提供了BeanFactoryPostProcessor让我们在生成BeanDefinition之前修改BeanDefinition的定义。
+ 了解Bean的一生：根据BeanDefinition结合CgilibSubclassingInstantiationStrategy初始化子类代理实例后，实例被包裹成BeanWrapper实例。查看Bean是否实现了各种Aware接口，将对应的实例注入到当前Bean实例中。执行BeanPostProcessor前后置处理，其实Aware接口就是执行BeanPostProcessor前置处理解决的。

## 第5章 Spring IoC容器ApplicationContext

ApplicationContext额外提供的功能：
+ 统一资源加载策略：ResourceLoader和Resource、ResourcePatternResolver批量查找资源。
+ 国际化信息支持：格式化包括货币格式、时间的表现形式、语言文字等。Java中的国际化主要有Locale和ResourceBundle两个类，Spring则提供了MessageSource来支持国际化
+ 容器内部事件发布：ApplicationEvent是事件，而ApplicationListener则会被容器自动识别并监听ApplicationEvent事件。ApplicationContext也是一个ApplicationEventPublisher。
+ 多配置模块加载的简化

ApplicationContext扮演的角色：
+ BeanFactory
+ ResourceLoader统一资源加载
+ MessageSource国际化支持
+ ApplicationEventPublisher事件发布器

@Component和@Bean的区别：@Component并不能加在第三方类库的类上，这时需要使用@Bean注解实现这个功能，但@Bean注解只能使用在@Configuration下。

## 第7章 一起来看AOP

我们可以把业务需求实现为Class类型的积木块，而系统需求(日志、权限检查)实现为Aspect类型的积木块。我们统称实现AOP的语言为AOL，AspectJ就是拓展自Java的AOL。

Java平台上的AOP实现机制：
+ 动态代理：基于接口，Spring AOP默认使用这种。
+ 动态字节码增强Cglib：生成源类的子类，程序运行时使用的是这些子类。缺点是如果源类是final的，则无法继承。

AOP的概念：
+ JoinPoint：连接点，被PointCut切分后形成的连接点
+ PointCut：切入点，可以使用正则表达式等声明切入的地方，切分后形成多个连接点
+ Advice：织入到PointCut的横切逻辑
+ Aspect：是PointCut和Advice的封装。

织入：JBoss AOP是使用自定义类加载器实现，Spring AOP使用一组类来实现。

### Spring AOP概述及其实现机制

Spring AOP属于第二代AOP，采用动态代理机制和字节码生成技术实现。

Spring AOP仅支持方法拦截的JoinPoint，但这已经能满足80%的需求了。所以我们不必担心Spring AOP的能力。

在Aspect(Spring的Advisor是特殊的Aspect)中设置好PointCut和Advice后，就需要把他们织入了。Spring AOP中使用的是ProxyFactory作为织入器。

从Pointcut到Advice再到Advisor，从目标对象到相应的代理对象，全部都由IoC容器统一管
理。为ProxyFactoryBean指定目标对象、要代理的接口类型以及相应的Advisor或Advice，
ProxyFactoryBean就会返回目标对象的代理对象供调用者使用。我们可以将生成的代理对象直接注入到依赖的主体对象中。

对一个个的目标对象编写ProxyFactoryBean太繁琐了，可以使用自动代理，即结合BeanPostProcessor实现。

### Spring AOP二世

@Aspect封装@PointCut和@Advice

```java
@Aspect
@Component
public class DemoAspect {

    @Pointcut("execution(public String *.getName())")
    public void pointCut(){};

    @Around("pointCut()")
    public Object executePointCut(ProceedingJoinPoint joinPoint) throws Throwable{
        System.out.println("开始执行");
        Object proceed = joinPoint.proceed();
        System.out.println("结束执行");
        return proceed;
    }
}
```

## 第四部分 使用Spring访问数据

统一异常访问体系：例如SQLException，具体的异常由各个数据库提供商来定义，无法统一。Spring将检查异常catch并转为RuntimeException。实现了统一异常访问。

基于Template的JDBC使用方式：统一封装了异常，使用模板方法封装了相同的代码块。

Spring对各种ORM的集成

## 第五部分 事务管理

+ 全局事务：分布式事务
+ 局部事务：单数据源事务

Java平台的局部事务支持：通过数据库连接Connection的自动提交设置为false，开启手动事务。如果使用的是Hibernate，就得使用Session管理事务
Java平台的分布式事务支持：
+ 基于JTA的分布式事务管理
+ 基于JCA的分布式事务管理

### 第六部分 Spring的Web MVC框架

Spring MVC框架的主要角色和流程如下：
DispatcherServlet前端控制器分发请求 => HandlerMapping映射对应的处理器比如Controller => Controller处理具体的业务 => ModelAndView模型和视图 => ViewResolver视图解析器 => View视图

![tVmjUO.png](https://s1.ax1x.com/2020/05/28/tVmjUO.png)

更细致一点，Handler并不只有Controller，如下图：

![tVnS8H.png](https://s1.ax1x.com/2020/05/28/tVnS8H.png)

在Spring MVC中任何可以用于web请求处理的处理对象统称为Handler，Controller只是Handler的一种。