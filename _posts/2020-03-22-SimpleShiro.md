---
layout: post
title: 最简明的Shiro教程
date: 2020-03-22
tags: 新技术
---

# 最简明的Shiro教程

后端管理系统登录一般都涉及到权限控制，权限管理组件用的最多的就是Apache的Shiro了，任何系统的登录模块，基本都可以使用shiro来实现我们的功能。

## 什么是Shiro

相信看到这篇文章的人都知道Shiro是什么吧，Apache Shiro是Java的一个安全（权限）框架，Shiro可以非常容易的开发出足够好的应用，JavaSE和Java EE环境都可以使用。Shiro可以完成：认证、授权、加密、会话管理、与web集成、缓存等。官方图如下：

![8RIiRK.png](https://s1.ax1x.com/2020/03/21/8RIiRK.png)

这里说下shiro安全的四大基石（上图中上层4个黄色部分）
+ Authentication：认证，身份验证，有时称为“登录”，这是证明用户“我是谁”的行为。
+ Authorization：授权，访问控制的过程，即确定“谁”有权访问“什么”。
+ Session Management：会话管理，管理特定于用户的会话，即使在非web或EJB应用程序中也是如此。
+ Cryptography：加密，使用密码算法保持数据安全，同时仍然易于使用。

## 使用shiro

shiro的使用也很简单，如下图，主要有三个类
+ Subject：主体，代表当前用户，当前登录用户的所有属性都可以通过该类获取
+ SecurityManager：安全管理器，判断Subject是否能登录、是否具有某些权限等等操作
+ Realm：数据访问器，shiro通过Realm访问数据库获取权限信息

![8RTp36.png](https://s1.ax1x.com/2020/03/21/8RTp36.png)

三者在代码中的体现是：
简单的说，当你对当前登录用户Subject进行操作时，Subject会与SecurityManager交互，SecurityManager会使用Realm查询权限。

当然，还有另外一个类ShiroFilterFactoryBean也比较重要，下面讲。

## Spring Boot整合Shiro

话不多说，上代码。首先来思考一下正常的网页登录流程是什么样的？我整理的一下，大概流程如下。
+ 页面拦截的设置，系统中有些页面可以直接访问，有些页面需要登录才能访问，有些页面不仅需要登录还需要登录的用户有相关权限才能访问
+ 如果用户没登录直接访问需要登录或需要权限的页面，则调转到登录页面让用户登录
+ 用户登录认证后，访问需要权限的页面时，系统可以识别该用户是谁并鉴权。

如果解决以上三个问题，上代码。

## 导入shiro-spring-boot-web-starter
maven中导入如下Shiro的坐标。

```
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>1.5.1</version>
</dependency>
```

## 拦截所有页面

步骤都在下面代码的注释了，总结一句话就是：通过ShiroFilterFactoryBean配置拦截的页面，拦截后调转到登录页，登录过程中，SecurityManager调用Realm来获取认证、授权的信息。

```java
package io.github.zebinh.zmall.mall.admin.config;

import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * Shiro配置类
 */
@Configuration
public class ShiroConfig {


    /**
     *  Shiro自带的过滤器，可以在这里配置拦截页面
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(@Autowired DefaultWebSecurityManager securityManager){

        // 1. 初始化一个ShiroFilter工程类
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();

        // 2. 我们知道Shiro是通过SecurityManager来管理整个认证和授权流程的，这个SecurityManager可以在下面初始化
        shiroFilterFactoryBean.setSecurityManager(securityManager);

        // 3. 上面我们讲过，有的页面不需登录任何人可以直接访问，有的需要登录才能访问，有的不仅要登录还需要相关权限才能访问
        Map<String, String> filterMap = new LinkedHashMap<>();

        // 4. Shiro过滤器常用的有如下几种
        // 4.1. anon 任何人都能访问，如登录页面
        filterMap.put("/api/user/login", "anon");
        // 4.2. authc 需要登录才能访问，如系统内的全体通知页面
        filterMap.put("/api/user/notics", "authc");
        // 4.3. roles 需要相应的角色才能访问
        filterMap.put("/api/user/getUser", "roles[admin]");
        // 5. 让ShiroFilter按这个规则拦截
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);

        // 6. 用户没登录被拦截后，当然需要调转到登录页了，这里配置登录页
        shiroFilterFactoryBean.setLoginUrl("/api/user/login");
        return shiroFilterFactoryBean;
    }

    /**
     * SecurityManager管理认证、授权整个流程
     */
    @Bean
    public DefaultWebSecurityManager defaultWebSecurityManager(@Autowired UserRealm userRealm){

        // 7. 新建一个SecurityManager供ShiroFilterFactoryBean使用
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        // 8. SecurityManager既然管理认证等信息，那他就需要一个类来帮他查数据库吧。这就是Realm类
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    /**
     * 自定义Realm，当SecurityBean需要来查询相关权限信息时，需要有Realm代劳
     */
    @Bean
    public UserRealm userRealm(){
        return new UserRealm();
    }


    /**
     * 为了方便观看，我将UserRealm类的实现写在这里了
     */
    class UserRealm extends AuthorizingRealm {

        // 9. 前面被authc拦截后，需要认证，SecurityBean会调用这里进行认证
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
            System.out.println("执行认证逻辑");
            UsernamePasswordToken token = (UsernamePasswordToken)authenticationToken;

            // 9.1. 为了方便演示，我这里写死了用户admin-name密码admin-pwd才能登录
            if (token.getUsername() == null || !token.getUsername().equals("admin-name")){
                return null;
            }

            return new SimpleAuthenticationInfo("", "admin-pwd", "");
        }

        // 10. 前面被roles拦截后，需要授权才能登录，SecurityManager需要调用这里进行权限查询
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
            System.out.println("执行授权逻辑");

            // 10.1. 为了方便演示，这里直接写死返回了admin角色，所有能登录的角色都是admin了
            SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
            info.addRole("admin");
            return info;
        }
    }

}

```

调转到登录页后，用户登录，直接调用Subject类的login方法即可，Subject类会调用SecurityManager进行认证。看到这里，应该就能理解本文开头的两幅图了吧。
```java
package io.github.zebinh.zmall.mall.admin.user.controller;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.UnknownAccountException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.subject.Subject;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequestMapping("api/user")
public class UserController {

    @PostMapping("login")
    public String login(@RequestBody Map<String, String> user){

        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(user.get("username"), user.get("password"));

        try{
            subject.login(token);
        }catch (UnknownAccountException e){
            System.out.println("用户名不存在");
            return "用户名不存在";
        }catch (AuthenticationException e) {
            System.out.println("认证失败");
            return "认证失败";
        }
        return "登录成功";
    }
    
    @GetMapping("getUser")
    public Object getUser() {
        return SecurityUtils.getSubject();
    }

    @GetMapping("notics")
    public String notics(){
        return "通知";
    }
}

```

## Shiro如何识别用户

在登录完成后，shiro会发一个cookie给客户端，包含了sessionId，因此能识别当前用户，当访问需要登录的页面时，如发现cookie中有sessionId，判断该用户已登录，则不需要调用Realm再进行认证了。cookie如下所示。

![8hXc5T.png](https://s1.ax1x.com/2020/03/21/8hXc5T.png)

那么问题来了，cookie只有pc端才有，移动端并没有cookie，该怎么解决移动端的shiro认证问题呢，同时这种方式无法解决单点登录的问题，应为cookie只能在同源的网站被发送到服务器。

解决方案就是，这时候就要定义自己的Session管理器了。用户登录后，自定义Session管理器生成session并生成一个token给到前端，前端每次访问都传递token，shiro会调用自定义的Session管理器做校验。Session中可以放置当前登录用户的信息并放置在redis中。自定义的Session管理器每次从redis中读取数据即可。

## JWT(JSON Web Token)

上面我们说到了使用token + redis来存储用户信息。此次用户信息是保存在服务器的redis中的，还有另外一种方式JWT，它是将客户信息加密后返回给客户端，客户端每次使用该加密字符串访问服务器，服务器加密即可获得用户信息。这种方式则是将用户信息存放在客户端了。可以了解一下这种方式。

本篇文章讨论的只是Shiro的使用，对于设计一个安全的登录(单点)系统还需要很多讨论。

以上。