---
layout: post
title: 永久免费花生壳内网穿透 - 公网访问内网服务
date: 2020-03-21
tags: 新技术
---

# 永久免费花生壳内网穿透 - 公网访问内网服务

相信大多数程序员都有这个需求——特别是前端程序员，当写简历的时候，希望能贴上自己的个人作品网址供面试官看看效果，但是却不想每年花大几百买服务器部署自己的项目，有没有合适的办法解决这个问题呢。有的，那就是让公网访问自己的电脑上的项目不就行了，这就是内网穿透。

下面就图文教程手把手教你设置吧，非常简单。

## 下载花生壳软件

首先第一步就是下载并安装花生壳软件了，地址：(https://hsk.oray.com/download/)，它能帮助我们内网穿透。

![UTOOLS1584629123515.png](https://user-gold-cdn.xitu.io/2020/3/19/170f341985955234?w=1920&h=914&f=png&s=472063)

## 配置花生壳

安装完成后可以微信扫描关注它的公众号，登录花生壳软件。接着点击软件右下角的加号图标进入如下图的配置。

![UTOOLS1584629251279.png](https://user-gold-cdn.xitu.io/2020/3/19/170f3437b94596aa?w=1100&h=675&f=png&s=106307)

一般我们的网站都是http项目吧，应用类型选择http，这时会叫你支付6块钱的认证费。6块钱是认证费，软件是永久免费的，放心用吧。

内网主机使用ipconfig查下你的ip，一般是192.168开头的ip。所有信息填完后保存。

## 访问我们的项目

OK，此时在本地启动你的项目，我的是vue项目，使用```npm run serve```启动。

保存花生壳的信息后如下图，那个链接就是你的公网域名映射了。使用任何设备访问你的项目吧。

![UTOOLS1584629700658.png](https://user-gold-cdn.xitu.io/2020/3/19/170f34a57048e658?w=1100&h=675&f=png&s=77518)

我的项目链接如下：http://30h2p28468.qicp.vip/

欢迎大家尝试访问，我的电脑是基本不关机的。(ps：vue本地开发环境的文件比较大，首次访问的我网页需要比较长的时间加载，vue打包的一个文件有7M大)。需要注意的是，我们的免费版花生壳一个月只有1G的流量哦。

![UTOOLS1584629893763.png](https://user-gold-cdn.xitu.io/2020/3/19/170f34d4c58931f4?w=1517&h=780&f=png&s=74505)


以上，有什么不懂的可以留言或私信我帮你解决哦。