---
layout: post
title: Postman神操作总结：一分钟写带登录态的爬虫
date: 2020-05-28
tags: 工具
---

# Postman神操作总结：一分钟写带登录态的爬虫

最近发现的一个Postman神操作，借助Postman的导入请求功能，一分钟不到、并且一行代码都不用写就能实现带登录态的爬虫了，这篇文章算是对web的各种工具的综合运用的总结吧。

废话不多说，我就以掘金的沸点热门为例子，来抓取所有热门的帖子吧。

![te64B9.png](https://s1.ax1x.com/2020/05/28/te64B9.png)

## 1. 分析请求获取api地址

如下图所示，打开chrome的F12分析到掘金沸点的数据接口。在api地址上右键 => Copy => Copy as cURL(bash)，第一步结束。简单吧。

![teccVA.png](https://s1.ax1x.com/2020/05/28/teccVA.png)

## 2. 上Postman神器

按下图操作，导入请求。

![teg4F1.png](https://s1.ax1x.com/2020/05/28/teg4F1.png)

请求导入了之后，手贱点击Send按钮测试一下，发现返回来的数据没问题。

![teRpNR.png](https://s1.ax1x.com/2020/05/28/teRpNR.png)

见证奇迹的时刻，如下图，点击Code，各种代码的请求都给你写好了，因为我是Java程序猿，这里就使用OkHttp来举例吧，复制这里的OkHttp请求代码。前端的同学可以选择JavaScript。前端到这里就结束，不用往下看了，后面的事情大前端的同学都懂的。

![teWiIs.png](https://s1.ax1x.com/2020/05/28/teWiIs.png)

## 3. 安装IDEA的GsonFormat插件

为了践行一行代码都不写就写出一个爬虫，这里要根据json反向生成JavaBean类。如下图，安装GsonFormat插件，重启IDEA生效。

![teW4Fs.png](https://s1.ax1x.com/2020/05/28/teW4Fs.png)

如下图，复制Json结果。

![tefATH.png](https://s1.ax1x.com/2020/05/28/tefATH.png)

新建一个Maven项目，新建一个类存放Json结果，暂且叫JueJin吧。

![tefxHg.png](https://s1.ax1x.com/2020/05/28/tefxHg.png)

接着将在Postman复制的Json结果粘贴进去，点击OK，接下来的弹窗也是直接点击OK即可生成JueJin类了。

![teh8KK.png](https://s1.ax1x.com/2020/05/28/teh8KK.png)

接着写一个main方法，粘贴我们上面复制的OkHttp的请求代码。

![te4cy6.png](https://s1.ax1x.com/2020/05/28/te4cy6.png)

## 4. maven导入jar包

这里在pom.xml文件中导入OkHttp和Gson的jar包。就可以将main方法中找不到包的报红解决了。

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.7.2</version>
</dependency>
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
```

## 5. 写两行代码

直到这里一行代码都还没写过，就已经能获取到沸点的数据了。接着我们使用GsonFormat生成的JueJin类，将掘金的内容单独提取出来吧。如下添加json转bean的代码。如果获取出来只有一行的小伙伴，记得把参数里的1改大一点。

![te7XJe.png](https://s1.ax1x.com/2020/05/28/te7XJe.png)

结果如下图：

![teHVzj.png](https://s1.ax1x.com/2020/05/28/teHVzj.png)

## 6. 结尾

大家爬取的时候记得注意爬虫礼仪，不要频率过快把网站搞奔溃了。同时如果其他网站需要登录态登录的话，你可以自己先登录在复制Postman的请求代码。登录后，爬虫的行为就是代表你的行为了，很容易被后台监控到，且行且珍惜。