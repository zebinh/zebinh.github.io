---
layout: post
title: 项目中框架源码debug跟踪
date: 2020-04-27
tags: 问题解决
---

# 项目中框架源码debug跟踪

最近在项目中遇到了一个多数据源无法切换的问题，项目的框架使用的是Mybatis + Hibernate + baomidou/dynamic-datasource-spring-boot-starter(github开源项目)，框架的思想是使用Mybatis编写sql语句，利用Mybatis拦截器(Mybatis叫做插件，其实就是一个Interceptor类)将sql语句拦截转Hibernate来执行。dynamic-datasource是baomidou开源的动态数据源切换的项目。

问题出现：dynamic-datasource使用@DS注解在方法上就能声明式的切换数据源，使用过程中发现，如果在同一个线程中的两个方法使用了不同的@DS声明，则无法顺利切换数据源，永远使用的是该线程拿到的第一个数据源。

问题分析：有可能是Hibernate对于以线程的级别管理数据源，但拿到第一个数据源时就hold住了，当该线程第二次要使用数据源时如果已有数据源则使用原来的数据源。

是否如我想的一样，只能自己debug Hibernate源码了。

## 配置和复现

项目的配置如下，默认使用的是slave数据源
![quicker_aa503b99-3f59-49d7-8083-53d90fd82c5a.png](https://i.loli.net/2020/04/26/YnQO2WeqsliyjfF.png)

正常来说，按照配置和我下面代码的执行顺序，数据源依次是slave => master => slave => slave，然而，中间的master无法切换，全程使用了slave数据源。如果把master放在第一位即 master => slave => slave => slave，则全程使用了master数据源。
![quicker_2269a9f7-409a-4774-8c39-80656a5d6efc.png](https://i.loli.net/2020/04/26/lBDcy46OxH8wLMP.png)

## debug

### debug经验
debug经验提一句，如下图
+ 先下载Maven依赖包的源码，才能跟踪Hibernate的源码，虽然idea已经有了自动反编译，但是反编译的代码是没有注释的，这样会使debug过程中行号对不上。导致看到的执行代码行并不是真正执行的行
+ debug中的按钮Drop Frame真的很好用
+ 最好把debug栈窗口和Console窗口通知打开，方便观察
+ Variable窗口可以随时修改你的变量值
+ 调试窗口的Get Thread Dump按钮可以打印当前的线程栈

下载源码：
![JgKbtS.png](https://s1.ax1x.com/2020/04/26/JgKbtS.png)

窗口调试
![quicker_38b6737e-e66e-43ee-b54d-e19ed53c6489.png](https://i.loli.net/2020/04/26/Y4a6zAHdWjqQ7nN.png)

### 开始debug
打断点，并使用post调接口，可以进入到断点，F7 step into一步步分析。这里
![quicker_759d855e-0564-444f-bcb9-66c1e1403ddb.png](https://i.loli.net/2020/04/26/4g8pqRnWdaxvCuM.png)

找了半天找到了执行获取结果集的代码，执行所有代码，重新调用接口
![JgJjeJ.png](https://s1.ax1x.com/2020/04/26/JgJjeJ.png)

再找到Connection的设置时机
![quicker_e0f5abab-0573-44be-92cc-c0b1d1e09ca3.png](https://i.loli.net/2020/04/26/lIMu5xHRCoNdabB.png)

接着找到了这里的判断，修改了变量值为null发现进入后数据源切换了，进一步验证了我的想法的正确性
![quicker_cc4e1674-d98a-4cbe-b113-c410edc7ce9f.png](https://i.loli.net/2020/04/26/P6mMJLCnfKUXYdT.png)

点击调试面板左侧的Get Thread Dump，将Connect断点处和开始执行Sql代码断点处的堆栈，可以看到分化处就是我们要找的地方了。
![quicker_29252ccd-934c-432b-a5a6-7b17a831d41a.png](https://i.loli.net/2020/04/26/i1vB6fxh4SFJPn7.png)

如图，顺着代码找到这个地方之后，我要修改==符号为!=让代码走到if块中，可以新建一个同包名同类名的文件。
![quicker_4e1fa8d8-5fda-4744-8270-43cbf7c2bcc9.png](https://i.loli.net/2020/04/27/SNU9QoOZ5YmGrX7.png)

顺着代码找，就能找到一个配置。更改该配置即可。
![quicker_e972c1cf-d3cd-4b38-a69e-f18785cb1f66.png](https://i.loli.net/2020/04/27/QfStb3nc6GMxmsL.png)


以上，本篇结束。