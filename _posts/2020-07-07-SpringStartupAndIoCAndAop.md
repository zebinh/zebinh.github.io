---
layout: post
title: 从Spring启动过程来理解IoC、AOP和bean的生命周期
date: 2020-07-07
tags: Spring
---

# 从Spring启动过程来理解IoC、AOP和bean的生命周期

Spring的基本功能就是IoC和AOP，我们的bean都是交给Spring管理的。那么Spring IoC是怎么生成这些bean、又怎么为指定的bean进行AOP代理增强呢？答案就在Spring的启动流程中。

## 1. Spring IoC

### 1.1. 鸟瞰Spring IoC

为了方便，这里使用注解版的写法来启动Spring IoC容器。如下图。

![quicker_318594ba-8017-4221-89af-6ac25ef672b2.png](https://i.loli.net/2020/07/04/OU1Ha3ncswj8NXW.png)

这里先说总结再讲解源码，先理清脉络再深入细节才不会迷失在细节当中。

![quicker_c37fdb93-6ae9-46f0-be83-820a17c6a783.png](https://i.loli.net/2020/07/04/sBEaZjmfq91tY7c.png)

如上图所示，Spring的启动过程主要可以分为两部分：
+ 第一步：解析成BeanDefinition：将bean定义信息解析为BeanDefinition类，不管bean信息是定义在xml中，还是通过@Bean注解标注，都能通过不同的BeanDefinitionReader转为BeanDefinition类。
    + 这里分两种BeanDefinition，RootBeanDefintion和BeanDefinition。RootBeanDefinition这种是系统级别的，是启动Spring必须加载的6个Bean。BeanDefinition是我们定义的Bean。
+ 第二步：参照BeanDefintion定义的类信息，通过BeanFactory生成bean实例存放在缓存中。
    + 这里的BeanFactoryPostProcessor是一个拦截器，在BeanDefinition实例化后，BeanFactory生成该Bean之前，可以对BeanDefinition进行修改。
    + BeanFactory根据BeanDefinition定义使用反射实例化Bean，实例化和初始化Bean的过程中就涉及到Bean的生命周期了，典型的问题就是Bean的循环依赖。接着，Bean实例化前会判断该Bean是否需要增强，并决定使用哪种代理来生成Bean。
    

Bean的生命周期如下图，先有个印象即可，到源码部分再回过头来看看Bean的生命周期。

![quicker_6161358d-a49c-4cfb-ac29-14d1f89ce689.png](https://i.loli.net/2020/07/05/AnMkBGIDtHbYf1u.png)

### 1.2. Spring IoC 源码

我现在用的是Spring 5.2.6的源码，Spring全注解版开发。

第一步就是new一个容器了。

![quicker_dfe02bf9-d87d-45bf-8108-1db3fa5e4a50.png](https://i.loli.net/2020/07/05/JZOMcHlV93PWtB8.png)

点进去看一下，可以看到主要有三个方法，请记牢这三个方法，this(); register(componentClasses); refresh();

![quicker_767126df-de5f-4aae-8864-ed51e35fc2bb.png](https://i.loli.net/2020/07/05/yM1iEChdgY6zDQl.png)

#### 1.2.1. this()方法

点击进入this，看到里层注册了6个RootBeanDefinition，即系统级别的BeanDefinition。

![quicker_c42e9e89-e1a6-4185-a6b7-c204b39eb649.png](https://i.loli.net/2020/07/05/Aboi6UEsTSze8Bm.png)

再进去，可以看到注册BeanDefinition其实就是放到BeanFactory的缓存中。

![quicker_5b940fba-094c-45d3-a684-b4d691be33ff.png](https://i.loli.net/2020/07/05/7glc9bshmriw3FP.png)

以ConfigurationClassPostProcessor类为例，其实它是一个BeanFactoryPostProcessor拦截器。注意，这部分回调的代码在refresh()中才会执行的。所以下面说的BeanFactoryPostProcessor还不会执行，而是在refresh()中执行。

![quicker_a74733ee-11af-438c-8377-b91177835edc.png](https://i.loli.net/2020/07/05/l3xZWVM7iwSm61f.png)

ConfigurationClassPostProcessor他是拦截配置类并解析里面的Bean定义的。其拦截方法会检查该类是否是配置类。

![quicker_e0e266c6-2042-4d50-8f45-19ae1e9d0011.png](https://i.loli.net/2020/07/05/bfLMmgOTSWqDRIu.png)

接着解析配置类。

![quicker_abe3cc16-aa13-4245-b08b-716a1ed2f7bd.png](https://i.loli.net/2020/07/05/bfRLyPUNBwv2391.png)

解析@Import和@Bean

![quicker_86e76306-b324-46bb-b32d-2748a564686c.png](https://i.loli.net/2020/07/05/4a2RkpolIYNQTym.png)

#### 1.2.2. register(componentClasses)方法

这个方法主要就是来注册new AnnotationConfigApplicationContext(xxxConfiguration.class);传进来的配置类的。

![quicker_74d24a9e-03e2-4940-b106-d94c5ee4bdb8.png](https://i.loli.net/2020/07/05/Gz3lOV9ZKcpAQfy.png)

### 1.3. refresh()方法

这是Spring启动中最重要的方法。点进去看一下。其中invokeBeanFactoryPostProcessor故名思意就是调用BeanFactory后置处理器。registerBeanPostProcessors(beanFactory)注册bean后置处理器，Bean后置处理器在Spring中应用很广泛，他能Bean创建过程中的拦截处理器，类似BeanFactoryPostProcessor也是拦截器。

![quicker_0eafea75-df6b-46c2-b3dc-8cb8a63518d7.png](https://i.loli.net/2020/07/05/TNDCOSrydMJjsmc.png)

点进这个方法，finishBeanFactoryInitialization(beanFactory)。他是初始化bean的重要方法。bean既可以通过@Bean来定义，也可以通过FactoryBean来初始化。

![quicker_ad92e8be-1532-4e78-a362-a873e04ae2d8.png](https://i.loli.net/2020/07/05/el2P4wCsjdUkctr.png)

点击getBean(beanName)看看一个bean是怎么创建的，同时，这也是Bean的生命周期。

![quicker_b43cf220-352b-4988-96e1-bd3208ee3daf.png](https://i.loli.net/2020/07/05/pkIHwKbDvOxyXEf.png)

看到createBean(beanName, mbd, args)方法

![quicker_36ae1662-7304-4c8b-ab0a-72489276ee37.png](https://i.loli.net/2020/07/05/tWZBzuslTF5XPmH.png)

bean创建过程可以分为两步，实例化Instantiation和初始化Initialization。实例化指的是创建bean实例，初始化指的是为填充bean实例属性（为属性赋值）。resolveBeforeInstantiation()方法在bean还没实例化之前执行。提供给Bean后置处理器一个返回代理的机会，当你调用被代理的bean时，实际上是执行了增强了的代理对象。

![quicker_b3dbfba4-72c8-43a8-b572-6663e5a7f084.png](https://i.loli.net/2020/07/05/dPT1wCXFyQHbB8G.png)

点进去doCreateBean方法。这里就是bean的生命周期了，如开篇放出的这张图。

![quicker_6161358d-a49c-4cfb-ac29-14d1f89ce689.png](https://i.loli.net/2020/07/05/AnMkBGIDtHbYf1u.png)

bean的生命周期，可以看到第一步就创建了实例

![quicker_1c9d022c-e97d-4814-80ce-8c46368d868f.png](https://i.loli.net/2020/07/05/hW7Y6bn2EckAivQ.png)

点击该方法createBeanInstance进去，可以看到最终就是通过Java的反射来创建bean对象的。

![quicker_04d3e466-1b00-4ac9-8e28-81dcba0edde8.png](https://i.loli.net/2020/07/05/E4Uw7RQFTV8Bkec.png)

点进去initializeBean方法查看，可以看到和上面生命周期的图吻合。先检查Aware接口，再到Bean后置处理器的前置处理方法，接着调用初始化方法。

![quicker_aca26d8a-b28c-4f6d-a92c-301857b4781f.png](https://i.loli.net/2020/07/05/YlZmXoVb5Q8iEFN.png)

bean的生命周期

![quicker_5640c1c9-7c55-4093-9a30-343c7e998ad9.png](https://i.loli.net/2020/07/05/NWCBPSu9gmhFpZH.png)

至此，bean的IoC容器功能启动流程讲解结束。

### Spring AOP

Spring代理增强方面，我们在上面的IoC容器部分看到实例化bean之前，其实是有先判断bean是否需要增强，该方法为resolveBeforeInstantiation(beanName, mdbToUse)，注意该方法是在bean实例化之前的，即先判断是否需要创建代理，如果不需要才会创建bean，否则创建的是代理对象。

夜深了，待续。明天再写写AOP部分的内容。

我来更新啦。

那么，Spring Aop是怎么创建代理的呢，我们来看下resolveBeforeInstantiation(beanName, bdbToUse)方法。

可以看到它调用的是InstantiationAwareBeanPostProcessor这个Bean后置处理器的方法，从类名字上看他是拦截Bean实例化阶段、而不是初始化阶段的。点进去看一下，可以发现他是通过调用InstantiationAwareBeanPostProcessor的回调方法来生成bean对象的。

[![UiomuT.png](https://s1.ax1x.com/2020/07/06/UiomuT.png)](https://imgchr.com/i/UiomuT)

调用postProcessBeforeInstantiation方法生成对象。

![Uios2t.png](https://s1.ax1x.com/2020/07/06/Uios2t.png)

至此，暂且打住。我们来看看我们一般是怎么使用Spring Aop的。

我们会写一个@Aspect注解的切面类，并使用@EnableAspectJAutoProxy注解启用代理。

![Uixh8S.png](https://s1.ax1x.com/2020/07/06/Uixh8S.png)

点进去可以发现，它导入了一个类AspectJAutoProxyRegistrar到Spring容器中。

![UFP9Nn.png](https://s1.ax1x.com/2020/07/06/UFP9Nn.png)

该类是一个ImportBeanDefinitionRegistrar类，搜索可以发现，ImportBeanDefinitionRegistrar类会在解析配置类的时候调用registerBeanDefinitions方法。

![UFPfCq.png](https://s1.ax1x.com/2020/07/06/UFPfCq.png)

该方法会向容器中注入一个AnnotationAwareAspectJAutoProxyCreator类的Bean定义。

![UFi7SP.png](https://s1.ax1x.com/2020/07/06/UFi7SP.png)

AnnotationAwareAspectJAutoProxyCreator类是一个InstantiationAwareBeanPostProcessor。

讲到这里，就和上面吻合了，实例化bean时现执行InstantiationAwareBeanPostProcessor，如果有返回对象，则使用该对象，否则才去创建实例。所以@EnableAspectJAutoProxy注解的作用就是向容器中添加一个InstantiationAwareBeanPostProcessor类，拦截bean的创建并生成代理对象。

![quicker_183097d7-606d-4d14-a9d5-896a174ed0d0.png](https://i.loli.net/2020/07/06/sXx7a9z3fylWpPj.png)

该拦截处理器如下，创建了一个代理对象并返回。

![quicker_6adc4364-70bd-4133-bcd9-133c1bf3bf95.png](https://i.loli.net/2020/07/06/CWbjhXkMxSHTZP5.png)

点进来createProxy(beanClass, beanName, specificInterceptor, targetSource)方法并跟踪下层代码，可以发现动态代理创建有两种方式，如果该类是接口，则使用JDK动态代理，否则使用的是Cglib代理。

![UFAprT.png](https://s1.ax1x.com/2020/07/06/UFAprT.png)

至此，Aop部分代码讲解结束。

其中，IoC的循环依赖问题和Aop的拦截器链设计有兴趣的可以推荐看一下源码。