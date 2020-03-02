---
layout: post
title: Java工程搭建方案 - Jenkins(Jar打包到docker)
date: 2020-03-03
tags: 工程
---

# Java工程搭建方案 - Jenkins(Jar打包到docker)

上一篇文章讲了使用Maven的聚合与继承构建微服务模块，并写了个helloworld demo在本地运行了起来，代码提交到了github上。

当然，这是远远不够的，如果每次修改了代码提交到github等源码仓库后，都需要运维人员checkout代码编译成jar包后运行，那是相当的繁琐。所以需要Jenkins这种CI自动化部署功能来解决这个问题。开发完成后，程序员交付的是代码，可以编译成jar包，但jar包需要运行在jvm下，所以更好的方案是，打包成docker交付。所以开发的交付产品演进是：代码 => Jar包 => docker镜像。

以下我来介绍下Jenkins自动化将代码编译打包成应用，并将应用打包到docker镜像。

## Jenkins CI自动化

Jenkins其实就是一个自动化工具，通过其自身的很多插件，可以帮你自动化的执行shell命令。所以我们的目标是，当我们提交代码到github后，登录我们jenkins平台，点击部署流水线，自动执行我们如下的任务：
+ 拉取github上我们的项目源码(git插件)
+ 使用maven构建编译打包，生成jar包(maven插件)
+ 把jar包发送到有docker环境的远程机器(scp)
+ 登录远程机器使用Dockerfile将jar包打包生成docker镜像(ssh插件)，docker run执行镜像
+ 部署成功发消息到钉钉(钉钉插件)

## 流程与步骤

以下简略的说明一些步骤。我使用的是一台CentOS 7虚拟机，上面安装有docker环境，Jenkins使用的是docker镜像Jenkins/Jenkins。运行时，Jenkins容器拉取github代码，并编译产生了jar包，因为Jenkins容器中没有docker环境所以无法将jar包打包成docker镜像，所以使用了scp命令将Jenkins容器中的jar包发送到CentOS虚拟机器，Jenkins中使用ssh插件登录CentOS虚拟机，执行Dockerfile将jar包打包成docker镜像并运行镜像。

## 遇到的问题

Jenkins插件下载报错？
+ 因为墙的存在，Jenkins很多插件下载不了，可以在插件管理中换源即可。

使用Jenkins容器？
+ 因为懒得配置Jenkins的环境，除了Jenkins外还需要安装git和maven，当然也可以让Jenkins帮你自动安装，使用docker镜像就省略了git安装，但docker镜像中没有maven。其实Jenkins就是一个web应用，下载其war包执行再使用浏览器登录即可。

在docker容器中使用root用户安装软件不知道密码？
+ docker运行这个命令以root用户登录，docker exec -it --user root <containerName> /bin/bash

因为为了方便，我把两个微服务放在同一个docker镜像中了，但docker的CMD命令只能执行一条语句
+ 可以使用&连接，如启动两个springboot可以写为java -jar demo1 & java -jar demo2，不能使用&&连接，因为&&会执行前面的程序成功才执行后面，而java -jar demo1是一个web包，永远处于服务状态无法返回成功，而&则同时执行了两条命令

## 成功

自此，写完代码提交到github后，登录本地搭建的Jenkins平台，点击部署流水线构建按钮，即可自动拉取代码，打包jar，打包成docker镜像，运行docker镜像。一键更新部署了代码，非常方便。


