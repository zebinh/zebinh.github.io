---
layout: post
title: Chrome插件SwitchyOmega配置
date: 2019-11-23
tags: 工具
---

# Chrome插件SwitchyOmega配置

作为一个只用三个键(ctrl+c, ctrl+v)写代码的码农，平时使用代理去用国外搜索引擎查找资料是必不可少的。以前不懂，都是打开代理之后就去修改系统的Internet选项，这样不仅麻烦不说，还将访问的http地址都代理了，这样访问国内的网址反而慢了。那是否有一个插件可以只能的判断网址是否需要代理，如果是国内的就直接访问即可？是的，没有chrome插件完成不了的事，这个插件就是SwitchyOmega。

## 原理

SwtichyOmega是chrome浏览器下的一款代理连接工具，它并不提供梯子，而是相当于维护了一张转发表，只是将浏览器中输入的地址转发到指定的端口，可以是本地端口或者远程服务器的端口。如下图所示：
![20191103000635.png](https://i.loli.net/2019/11/03/KLXNm2Ef3SsObhI.png)
上图已经说的很清楚，按步骤理解即可。

## 界面

SwitchyOmega初始界面如下图，点击红框处下新建一个情景模式
![20191103001628.png](https://i.loli.net/2019/11/03/D6qzQTsoc25nRW3.png)
我用的两个模式是如下两个模式，已经够用了。另外两个模式没研究过。
![20191103002242.png](https://i.loli.net/2019/11/03/3oKBTZdyl518cMf.png)

## 配置

### 代理服务器

如上界面的两张图，点击 新建情景模式 => 代理服务器，输入情景模式名称（这里是sock5），如下图。根据你的梯子的代理协议和端口(一般是1080)，填写完整即可。
![20191103002610.png](https://i.loli.net/2019/11/03/KkDYPeIaoUh2ORZ.png)
配置完成后，点击左下角的“应用选项”保存。到浏览器右上角点击选择你刚才建立的代理场景名称（这里是sock5），并开启你的梯子。
![20191103003237.png](https://i.loli.net/2019/11/03/k8CMrp5LltYWAjT.png)
自此，你输入的http地址都将转发到情景模式sock5，即本地的127.0.0.1:1080端口

### 自动切换模式

上面虽然可以代理了，但有个弊端，就是所有的http都做了转发，如果你访问国内的网址，也会转发，这样反而响应更慢了。这里我们可以新建另一种类型的情景模式，点击 新建情景模式 => 自动切换模式，输入情景名称（这里是auto-proxy），如下图操作。
![20191103004101.png](https://i.loli.net/2019/11/03/kQcVfUtiOAR43Y8.png)
设置为自动代理，如下图操作：其中，gwflist地址为：https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
![20191103004804.png](https://i.loli.net/2019/11/03/9tqDygkG5lnfa6x.png)
自此，可以自动代理了。

### 添加自定义的地址

上面虽然可以自动代理，但是也有一个弊端，就是github地址是不在gwflist中的，所以是直接访问的，但github直接访问速度太慢了，通过代理来访问反而会更快。那怎么添加到代理列表中呢。如下，在“自动切换模式”下新增一个域名通配符规则接口。
![20191103005957.png](https://i.loli.net/2019/11/03/Hb6gXvrchxTt8AM.png)
以上。