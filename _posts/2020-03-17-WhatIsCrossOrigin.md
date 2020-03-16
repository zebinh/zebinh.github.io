---
layout: post
title: 前端跨域问题
date: 2020-03-17
tags: 新技术
---

# 前端跨域问题

B/S架构的项目中前端经常会遇到跨域问题，什么是跨域问题，常用的解决方法又有哪些呢？可能大多数人对跨域问题都只是一知半解吧。

## 跨域问题的表现

先来说说那么到底什么是跨域？跨域是指一个域下的文档或脚本去请求另一个域下的资源。跨域问题则是指浏览器出于安全考虑而需要遵循同源策略，限制不同源的网站的文件的执行，同源指的是“协议+域名+端口号”都相同。

如果非同源，如下三种行为会受到限制：
+ 无法获取非同源网页的cookie、localstorage和IndexedDB
+ 无法访问非同源网页的DOM（iframe）
+ 无法向非同源地址发送AJAX请求（可以发送，但浏览器拒绝接受响应）

必须明确以下几点：
+ 跨域问题只存在于浏览器中，而在C/S架构中，如App中是不存在跨域问题的。
+ 浏览器的同源策略并不限制请求的发送，跨域时不同域的服务器是能收到请求的，但浏览器拒绝接受响应，如下图。

![UTOOLS1584368910617.png](https://user-gold-cdn.xitu.io/2020/3/16/170e3bf01f69c28a?w=797&h=377&f=png&s=37400)

那么为什么浏览器才有跨域问题而App不会存在跨域问题呢？很简单，app是自家的，所有的请求都是到自家的服务器，而浏览器是可以访问很多网站的，每个网站都可以带有cookie等信息，如果被其他恶意网站利用，后果不堪设想。

## 跨域解决方案

跨域的解决方案很多，有如下几种：

![UTOOLS1584371858797.png](https://user-gold-cdn.xitu.io/2020/3/16/170e3ebfa3584901?w=354&h=300&f=png&s=29196)

常用的方案有：
+ 跨域资源共享(CORS)
+ Nginx代理跨域
+ JSONP

### 跨域资源共享CORS

CROS全称是跨域资源共享(Cross-origin resource sharing)，它的提出就是为了解决跨域请求的。跨域资源共享(CORS)标准新增了一组 HTTP 首部字段，允许服务器声明哪些源站有权限访问哪些资源。

对于简单请求服务器配置后直接接受请求，对于对那些可能对服务器数据产生副作用非简单的HTTP请求方法（特别是GET以外的HTTP请求，或者搭配某些MIME类型的POST请求），浏览器必须首先使用OPTIONS方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨域请求。

我们上面说过，浏览器跨域时其实请求已经发送到服务器了并返回了结果，那如果服务器配置了是允许跨域的，浏览器是否就会放行呢？答案是肯定的，当浏览器拿到响应头```Access-Control-Allow-Origin: *```时，会匹配其中的允许跨域的网址是否有自己，有的话则接收响应执行。如下，我在服务端Spring Boot配置了注解，成功跨域。

服务端配置：当然你可以配置成全局的，不用每个类都写注解

![UTOOLS1584373379959.png](https://user-gold-cdn.xitu.io/2020/3/16/170e403305645dff?w=466&h=285&f=png&s=25081)

响应如下：

![UTOOLS1584373449072.png](https://user-gold-cdn.xitu.io/2020/3/16/170e4043e624c60d?w=971&h=477&f=png&s=53370)

### Nginx代理跨域

CORS除了上面简单使用Spring Boot注解配置外，还可以使用Nginx配置。如下，对于非简单请求，浏览器总共会发两次请求，第一次是预检请求OPTIONS。
```
location / {  
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';

    if ($request_method = 'OPTIONS') {
        return 204;
    }
} 
```


### JSONP

JSONP的原理是利用了<script>标签的src属性没有跨域的限制，每次需要跨域请求A地址时，动态生成一个<script>指向A地址并且带上回调函数名(例如名为callback)，服务端收到请求后使用该回调名拼接返回参数，如callback('helloworld')，成功返回后，浏览器将执行本地写好的callback方法。

JSONP需要浏览器和服务器配合。


## 参考资料
+ [什么是跨域？解决方案有哪些？
](https://cloud.tencent.com/developer/article/1175899)
+ [彻底理解浏览器的跨域
](https://juejin.im/post/5cad99796fb9a068ab40a29a)
+ [Nginx配置跨域请求 Access-Control-Allow-Origin *](https://segmentfault.com/a/1190000012550346)