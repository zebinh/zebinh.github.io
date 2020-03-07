---
layout: post
title: Node.js是什么
date: 2020-03-07
tags: 新技术
---

# Node.js是什么

什么是Node.js呢？一句话总结，Node.js是基于chrome V8 Javascript引擎基础上的一个库，使得Javascript(下称JS)脱离浏览器运行并提供了丰富的方法库。

我们知道，传统前端编写的JS代码都是运行在浏览器中的，那如果在服务器的黑白界面上，没有浏览器不就没法运行JS了？非也，只要有JS引擎即可运行JS代码，Node.js就是基于chrome V8 JS引擎的。同时，Node.js提供了事件驱动I/O等特性，使得你使用JS就可以编写事件驱动代码，因此又说他是一个库。

## Hello World

安装完Node.js环境后，打开记事本输入：
```js
console.log("Hello World");
```
保存为helloworld.js后，使用命令
> node helloworld.js

即可看到输出结果，不需要用浏览器。当然你也可以直接输入node命令进入交互模式，在交互模式里打代码。这样看起来JS脚本就和python脚本差不多了。

## 创建一个服务器

Node.js是应用于后端的，所以跟SpringMVC一样，我们可以用它写一个服务端应用，只需要引入Node.js提供的http模块，代码如下：
```js
var http = require('http');

http.createServer(function (request, response) {

    // 发送 HTTP 头部 
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, {'Content-Type': 'text/plain'});

    // 发送响应数据 "Hello World"
    response.end('Hello World\n');
}).listen(8888);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8888/');
```

接下来跟SpringMVC编写服务端应用的套路一样，使用框架提供的方法进行开发。