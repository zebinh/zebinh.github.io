---
layout: post
title: 写给后端的前端快速搭建笔记
date: 2020-08-10
tags: Vue
---

# 写给后端的前端快速搭建笔记

迫于平时学习了前端之后没有形成体系，并且我是做后端的，没打前端的代码会生疏，下次又不知道怎么搭建前端框架了。因此这里记录一个前端的搭建过程，并写一个能用就行的前端界面。这里按照前端工程化的搭建流程来梳理技术栈，以Vue + ElementUI为例。

+ NodeJS：NodeJS是前端工程化的重要支柱，他使得JS脱离了浏览器环境，使得JS可以像脚本一样在主机上运行。
+ npm：NPM是随NodeJS安装的包管理工具，类似后端的Maven，使用npm install命令可以安装当前目录下package.json需要的包，本地按照时存放在当前目录./node_modules目录，全局安装则安装nodejs的安装目录，按照NodeJS语法，可以使用require()引入NodeJs包。使用命令npm run dev可以运行package.json中scripts节点下的dev节点对应的命令，比如
```json
{
  // ...
  "scripts": {
    "build": "node build.js"
  }
}
```

使用npm run build就是执行命令node build.js。

同时开过程中使用npm install xxx --save中save选项表示会讲这个模块写入package.json带上生产，而--save-dev则只在开发环境。
+ npm下载过慢，建议使用cnpm
```js
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

## 创建项目

安装完NodeJS和npm后，就可以创建项目了。使用vue-cli脚手架来创建工程比较简单，所以先安装Vue脚手架吧。

```
npm install --global vue-cli
```

安装完后，输入vue命令看看是否安装成功。

接着使用安装完的vue脚手架，执行如下命令生成项目。

```
vue init webpack myproject
```

之后，执行npm install和npm run dev，安装必要的包和启动vue项目。

安装element-ui库并将其带上生产。

```
npm install element-ui --save
```

## 开始开发

### 导入文件进行显示

Vue每个文件都是一个组件，文件结构为：
```
<template>
</template>
<script>
</script>
<style>
</style>
```

文件融合了html、css、js，是一个完整的组件文件。因此开始写页面时，你可以在components目录下新建一个vue文件，然后在首页导入看到效果。其实component只是一段代码，注册到vue中即可使用。前提是你将文件导入进来。
+ 想注册一个组件
+ 导入组件
+ 在template中使用

![quicker_6212eaf1-25a1-4a76-a220-1d94cfba613f.png](https://i.loli.net/2020/08/09/2uMH5aAVoBr7slt.png)

```
<template>
    <div>
        <Test></Test>
    </div>
</template>
<script>
    import Test from './components/test.vue'
    export default {
        components: {
            Test
        }
    }
</script>
```

项目根目录下的src/main.js是程序入口，通用的组件可以在这里注册。比如element-ui组件。
```
import ElementUI from 'element-ui' //element-ui的全部组件
import 'element-ui/lib/theme-chalk/index.css'//element-ui的css
Vue.use(ElementUI) //使用elementUI
```

上面的通过在template中使用<Test></Test>引用组件，比较low了。不如用router前端路由来实现。


先导出Test组件。
```
<script>
  export default {
    name: "Test",
    data() {
      return {
        activeIndex: '1',
        activeIndex2: '1'
      };
    },
    methods: {
      handleSelect(key, keyPath) {
        console.log(key, keyPath);
      }
    }
  }
</script>
```
在App.vue中放置一个路由占位符<router-view></router-view>。

```
<template>
  <div id="app">
    <router-view/>
  </div>
</template>
```
然后在router/index.js中导入并配置它。
```
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Test from '@/components/test.vue'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/test',
      name: 'Test',
      component: Test
    }
  ]
})

```

### 页面布局

到了上面这一步，你就可以自定义添加自己喜欢的页面了。写前端最麻烦的就是布局，所以这里讲一下布局的使用。现在的布局主要使用flex弹性布局。

首先在外层div（以下称为容器）使用display: flex，其下的子元素默认会变为行内块元素，可以定义大小。如果想定义宽度是整个屏幕占比多少，可以使用width: 33vx，视口被均分为100份，取33份。布局一般宽度用占比，高度定死。

容器中有主轴和侧轴之分。默认主轴是横向的，侧轴是纵向的。子元素的排列就是通过调整主轴侧轴来实现的。

如果想改变主轴的方向，可以使用flex-direction: column设置主轴为垂直方向。

如果想调整主轴元素的间隔布局，可以使用justify-content
+ center：在主轴居中对齐
+ space-around: 平分剩余空间
+ sapce-between：两端对齐，再平分剩余空间

如果主轴元素过多，会导致元素被挤压在主轴上，可以使用flex-wrap: wrap让元素换行。

如果想让元素垂直居中，则需要调整侧轴的布局。align-items设置侧轴布局，适用于单行。
+ center：居中
+ stretch：拉伸，拉伸时子元素不能设置高度

当上面元素是多行的，则align-items不起作用。我们可以使用align-content来调整多行间的间距和布局。
+ center: 居中
+ space-around: 平分剩余空间
+ sapce-between：两端对齐，再平分剩余空间

上面都是容器的属性，接下来介绍一些子元素的属性。

如果你子元素有三个，前两个设置了宽度，想让第三个占所有剩下的宽度，可以使用flex属性。

element-ui的布局建议看视频：https://www.bilibili.com/video/BV1NK4y187XH?p=5

vue-router的路由视图<router-view></router-view>只能获取到顶层路由，即#App下的路由器，然后push进去路由地址，如果是嵌套路由，有点麻烦。

前端调用后台的接口可以使用axios模块，先安装再导入。
```
# 安装
npm install axios

# 导入并使用
import axios from 'axios'
Vue.prototype.$axios = axios
```
使用方法：
```
this.$axios
      .get("https://www.v2ex.com/api/topics/hot.json")
      .then(function(res){
        console.info(res);
      })
      .catch(function(err){
        console.info(err)
      })
```
上面会出现跨域报错，需要使用nginx反向代理解决。