---
layout: post
title: Java工程搭建方案 - Maven
date: 2020-02-24
tags: 工程
---

# Java工程搭建方案 - Maven

本篇讲讲Maven在实际项目中的应用，包含
+ Maven继承和聚合
+ 微服务项目划分
+ Maven仓库配置和顺序
+ Maven私服的配置

## 前言

从零开始一个项目，第一步自然是做系统模块划分了。

Maven是Java世界最流行的项目管理工具。其提供的继承和聚合功能，使得项目可以划分为多个模块。跟微服务的思想是很吻合的，是实现微服务工程很好的工具。

## 微服务和Maven聚合模块的区别

微服务是一个可以独立部署的工程，而Maven的聚合功能，可以将多个模块（工程）聚合成为一个大项目来管理，其本质还是一个一个单独的工程，不过通过Maven聚合后，可以一键在所有工程中执行相同的操作。举个例子，当在父工程中执行mvn clean命令后，Maven会查找其下的子工程一个个执行clean操作。

## 电商微服务系统

现在大多数系统，都有pc管理端界面和移动端界面(门户模块)，即后台管理系统和前台商城系统。还有各种商品模块、订单模块和用户模块。

+ 后端管理模块(openapi)
+ 门户模块(openapi)
+ 订单模块
+ 商品模块

具体怎么设计项目结构，还需要看不同的需求。

## 项目结构

### 单体应用

单体应用首先要考虑到的是代码复用问题，这个问题在拥有前台、后台的系统中更加常见，如查看商品列表接口，前后端都需要使用。

借鉴github上一个开源项目，其中的单体项目结构是比较合理的。

![UTOOLS1581761061286.png](https://user-gold-cdn.xitu.io/2020/2/15/170484e6004e0ffd?w=864&h=239&f=png&s=16169)

加入微服务的一些组件后如下，但并不能说它是一个正宗的微服务系统，因为像订单之类的业务模块并没有拆分。

![UTOOLS1581763942690.png](https://user-gold-cdn.xitu.io/2020/2/15/170487a5c23c72f7?w=772&h=319&f=png&s=29002)

### 微服务架构

我曾经待过一家做社交电商类的创业公司，其项目设计的理念十分的先进，微服务拆分得特别细。然而开发人员写起代码来却非常得复杂，因为其每一个微服务都新建了一个工程来维护。这样导致一个问题，如果一个开发人员开发的功能涉及到多模块时，需要同时打开多个idea窗口写代码。同时openapi端使用feign调用内部微服务接口时（如订单模块）需要再复制一份接口@RequestMapping到openapi端。

如下架构图所示，每个微服务都可以作为一个模块存在，对外的openapi不需要再包含子工程，而内部的微服务api需要分为api和provider两个子模块。以将api接口jar包提供给消费端。

![UTOOLS1582426999897.png](https://user-gold-cdn.xitu.io/2020/2/23/1706fffc81b49adf?w=556&h=540&f=png&s=29737)

通过实践，api子模块可以定义接口和@RequestMapping，provider模块的controller实现该接口即可提供服务。而消费端导入api子模块后，feign接口继承该api接口就可以实现调用。

## 项目搭建

我们就以简单的自建zmall工程为例说明idea搭建工程的步骤。github地址为：

结构为：
+ zmall（聚合父工程，pom工程，管理依赖版本）
    + mall-common（各模块通用的工具类）
    + mall-admin（openapi后台管理端接口）
    + mall-portal（openapi移动端页面接口）
    + mall-goods（商品模块，微服务，内网调用不对外）
        + mall-goods-api（商品模块微服务的接口定义，可打包成jar提供给消费端）
        + mall-goods-provider（商品模块业务实现）
    + mall-order（订单模块模块，微服务，内网调用不对外）
        + mall-order-api（订单模块微服务的接口定义，可打包成jar提供给消费端）
        + mall-order-provider（订单模块业务实现）

### 建立pom父工程

建立一个Maven项目，Maven坐标一般的定义为：
+ groupId：到项目级别，一般能明确公司和项目即可。如io.github.zebinh.zmall。以spring为例则为org.springframework
+ artifactId：到项目下的模块，一般使用短横线风格模块功能，如基础架构通用部分可以 用base-framework，业务部分可以加项目名，如mall-admin。以spring为例则为spring-core，spring-context等

建立完成后，POM的主要配置如下。作为统一项目管理。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- POM父工程主要统一管理子模块的依赖版本问题，体现Maven的继承和聚合功能 -->

    <!--  项目坐标  -->
    <groupId>io.github.zebinh.mall</groupId>
    <artifactId>zmall</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- spring-boot-starter-parent统一管理版本 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
    </parent>

    <!-- 统一管理属性和版本，这里的配置在启动时起作用，类似mvn -D参数传递-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <skipTests>true</skipTests>

        <swagger2.version>2.7.0</swagger2.version>
    </properties>

    <!-- 依赖管理 -->
    <dependencyManagement>
        <dependencies>

            <!-- SwaggerUI API文档生成工具 -->
            <dependency>
                <groupId>io.springfox</groupId>
                <artifactId>springfox-swagger2</artifactId>
                <version>${swagger2.version}</version>
            </dependency>

        </dependencies>
    </dependencyManagement>

</project>
```

### 建立子模块

以zmall-admin为例，建立一个spring boot工程，作为zmall的子模块。idea创建工程时记得修改以下几个点。

+ 新建工程时注意模块名转为包名时，加一个点号分隔。

![UTOOLS1581847583152.png](https://user-gold-cdn.xitu.io/2020/2/16/1704d76976782550?w=1287&h=856&f=png&s=39503)

+ 创建完后，需要修改pom文件的parent坐标等。

### 建立带有子模块的模块

类似mall-goods，其也是一个pom工程，其下有多个子模块。创建时需要注意以下问题。

+ 模块名加上短横线

![UTOOLS1581848171291.png](https://user-gold-cdn.xitu.io/2020/2/16/1704d7f90d176fed?w=1287&h=858&f=png&s=27331)

+ 使用spring initializer建子模块mall-goods-provider时，需要加长到子模块目录下

![UTOOLS1581848429732.png](https://user-gold-cdn.xitu.io/2020/2/16/1704d83828a26357?w=1287&h=858&f=png&s=29383)

+ 以上都需要修改pom文件

## Maven仓库和私服设置

Maven的用户配置文件.m2目录下的settings.xml一般配置如下几项：
+ localRepository：统一Maven本地仓库
+ servers：配置私服用户名和密码
+ repositories, pluginRepositories：私服或远程仓库配置，一般可以包含在profiles中做多配置，使用activeProfiles激活指定配置
+ profiles：配置多个，激活指定的一个
+ distributionManagement：使用mvn deploy时会部署到指定的私服，但是一般开发中不会指定。而是使用git下载代码mvn install安装到本地仓库。

当然，pom文件中也可以使用repositories配置仓库

maven仓库查找顺序：maven允许在pom文件和settings.xml文件中配置仓库的位置，其会根据maven仓库的id的字典序号来搜索仓库。因此如你只是临时使用阿里云仓库代替中央仓库，在pom.xml中配置id为a开头的阿里云仓库，这样就能排在前边解析了。


```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <!-- maven本地仓库位置 -->
    <localRepository>/develop/mavenRepository</localRepository>
    
    <!-- 私服用户密码，稳定版和快照版分开 -->
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>zebinh</username>
            <password>abcdefg</password>
        </server>
        <server>
            <id>nexus-snapshots</id>
            <username>zebinh</username>
            <password>abcdefg</password>
        </server>
    </servers>
    
    <!-- 使用!非，除了私服外，都使用阿里云仓库加速 -->
    <mirrors>
        <mirror>
            <id>mirror</id>
             <mirrorOf>!nexus-releases,!nexus-snapshots</mirrorOf> 
            <name>mirror</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
        </mirror>
    </mirrors>
    
    <!-- 多配置 -->
    <profiles>
        <profile
        
            <id>nexus</id>
            
            <repositories>
                <repository>
                    <id>nexus-releases</id>
                    <url>https://repo.aliyun.com/repository/1234-release/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
                <repository>
                    <id>nexus-snapshots</id>
                    <url>https://repo.aliyun.com/repository/1234-snapshot/</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
            
            <pluginRepositories>
                <pluginRepository>
                    <id>nexus-releases</id>
                    <url>https://repo.aliyun.com/repository/1234-release/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
                <pluginRepository>
                    <id>nexus-snapshots</id>
                    <url>https://repo.aliyun.com/repository/1234-snapshot/</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
            
        </profile>
    </profiles>

    <!-- 激活指定的配置 -->
    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>

</settings>
```

## 搭建项目骨架

理论指导依据：

![UTOOLS1582426999897.png](https://user-gold-cdn.xitu.io/2020/2/23/1706fffc81b49adf?w=556&h=540&f=png&s=29737)

各层的命名，如下为阿里巴巴Java开发手册定义：

![UTOOLS1582812175609.png](https://user-gold-cdn.xitu.io/2020/2/27/17086f51b3c4a965?w=1159&h=501&f=png&s=163847)

我的实践：
+ controller层：出参统一使用VO。入参如果为如果是增删改功能，使用DTO，入参如果为查询条件，使用QO(Query Object)
+ service层：入参可以为DTO，出参为BO
+ dao层：入参为字符串等，可以使用DTO，出参为DO，DO和数据库Entity对象一一对应，可能比Entity少一些主键等字段

### 首先搭建mall-goods-api

mall-admin和mall-portal是openapi，故其下的包需要分模块命名，如goods.controller、order.controller。而mall-goods-api是专门用于商品的微服务，故不需再分模块了。

api是接口层，提供给admin和portal调用，输出的是vo对象。因此其需要定义的是url路径和vo对象，mall-goods-provider模块实现api的接口。如下：

![UTOOLS1582428307274.png](https://user-gold-cdn.xitu.io/2020/2/23/1707013bb187c765?w=557&h=675&f=png&s=38210)

![UTOOLS1582428554306.png](https://user-gold-cdn.xitu.io/2020/2/23/17070177ff944c47?w=1357&h=461&f=png&s=79795)

### 接着搭建mall-admin openapi

上面搭好了goods微服务，mall-admin则可以引入mall-goods-api来调用goods模块了。如下图：

![UTOOLS1582431393011.png](https://user-gold-cdn.xitu.io/2020/2/23/1707042ddf369393?w=1311&h=618&f=png&s=74049)

至此，maven工程搭建结束。代码github地址：https://github.com/zebinh/zmall