---
layout: post
title: Netty源码服务器启动流程
date: 2020-08-15
tags: Netty
---

# Netty源码服务器启动流程

看到这篇文章的应该都用过Netty吧。Netty服务端的模板代码如下，我们分析下它是怎么启动的。不要纠结没有关闭连接的代码，毕竟我们只是用这段代码来debug。这篇文章我主要写的是Netty服务端的启动流程。

读完这篇文章你会知道：
+ Netty的几大组件的关系是什么？包括NioEventLoopGoup, NioEventLoop, Channle, ChannelHandler, Pipeline
+ Netty是怎么注册感兴趣的事件的？
+ Netty服务的bossGroup收到连接后，是怎么转派到workerGroup的。
+ Netty服务端启动时经历了什么？


```java
package netty.simple;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class NettyServer {

    public static void main(String[] args) throws InterruptedException {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        ServerBootstrap bootstrap = new ServerBootstrap();
        bootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 28)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childHandler(new ChannelInitializer<SocketChannel>() {

                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        socketChannel.pipeline().addLast(new MyChatHandler());
                    }
                });
        ChannelFuture sync = bootstrap.bind(6666).sync();
        sync.channel().closeFuture().sync();
    }
}

```

## EventLoopGroup事件循环组

Netty是基于Reactor事件模型的，它将通道的连接和处理分开，bossGroup负责处理NioServerSocketChannel通道连接，通道连接后生产NioSocketChannel分派到workerGroup处理通道的读写事件和业务逻辑。具体怎么通道怎么分配的，接下来再说明。

EventLoopGroup顾名思义，它就是EventLoop组，维护一堆EventLoop的，如下图(NioEventGroup实现了EventExcutor接口)。默认new EventLoopGroup()会生成(cpu核心数*2)个EventLoop，每个EventLoop绑定一个端口，所以如果服务只绑定一个端口的话，就指定为1个接口，即new EventLoopGroup(1);

![quicker_2d9ab9a7-deb0-4ef4-a230-7d549ef8ae5a.png](https://i.loli.net/2020/08/15/wA6FQzfLnRovpkT.png)

## NioEventLoop事件循环

每个EventLoop都有一个死循环，负责监听Channel上的事件。看NioEventLoop的继承链，它是一个EventExecutor、EventLoop，所以NioEventLoop是一个线程池。NioEventLoop的父类SingleThreadEventExecutor维护了一个线程池对象。

![quicker_e527dc0a-cd93-444a-83a9-b2a57938435e.png](https://i.loli.net/2020/08/15/hB3v4PaLSfMWTrJ.png)

NioEventLoop的execute(Runable r)方法，这是线程次的提交方法，当提交一个线程到NioEventLoop中时，就是执行这个方法，它做的工作如下图，是将提交过来的线程task入队，然后执行NioEventLoop的线程提交方法execute，先将task入队，再看情况执行startThread()。

![quicker_a7b027ff-e7f7-453d-a8a2-bcc170288b05.png](https://i.loli.net/2020/08/15/svARuniWrXDp51C.png)

跟踪进去，它会执行doStartThread()方法。这个方法提交一个线程，立马执行了SingleThreadEventExecutor.this.run()方法。这里，虽然NioEventLoop没有继承Thread或者实现Runnable接口，不能认为它是一个线程实现类。但是它有自己的run()方法，并且初次启动NioEventLoop时会执行SingleThreadEventExecutor.this.run()方法，因此暂且可以认为它是一个线程吧。

![quicker_db1d3441-3d5a-4923-b202-6237c5fca185.png](https://i.loli.net/2020/08/15/ZOLRkfdmPHiuMBI.png)

那NioEventLoop这个名义上的线程它的run方法是什么呢，如下图，他就是一个死循环了。说到这里，就知道问什么我们说NioEventLoop是一个死循环的线程了吧。

![quicker_6f3f0e5b-fd81-458c-b9eb-42e4bb783504.png](https://i.loli.net/2020/08/15/924bJaVnxhszULv.png)

## Channel通道

再看到ChannelFuture f = b.bind(PORT).sync();这一句的bind方法，它做了初始化和注册通道的任务。即ServerBootstrap初始化了注册了通道。

![quicker_95a2ed91-b18c-4556-bab1-2ef01443a210.png](https://i.loli.net/2020/08/15/kumZJ86LDTQxi2g.png)

他会根据.channel(NioServerSocketChannel.class)这句代码生成一个服务端的NioServerSocketChannel。

![quicker_425eaf88-3f90-4126-b4e2-c2b6a09cd334.png](https://i.loli.net/2020/08/15/AB5SF3knwxVGuXC.png)

Channel初始化时，会给Channel塞很多东西，比如感兴趣的OP_ACCEPT事件，和非阻塞的配置，看到这里就明白了Netty底层还是NIO，Netty其实就是对NIO的封装了。

![quicker_a67418a8-0745-4e71-8504-9b2efb8f7d01.png](https://i.loli.net/2020/08/15/ZrzDPMW7GVKcspm.png)
![quicker_096e46c5-2e44-4ae8-8fde-eaa49fdcd93f.png](https://i.loli.net/2020/08/15/bIWaPZAjVoUBfDS.png)

还向该Channel的pipeline中加入了一个Channel初始化器ChannelInitializer，那么ChannelInitializer的initChannel什么时候执行呢？先剧透一下，这个方法在Channel注册完成后执行，现在还在Channel初始化阶段，还没执行。

![quicker_a31b15d8-0906-431d-91b4-b38e9065eaba.png](https://i.loli.net/2020/08/15/LlbsyNeatpJPYgF.png)

## Pipeline管道

写过Netty小demo的都知道，我们需要处理自己的业务逻辑时都是向Channel对应的pipeline中加入我们的ChannelHandler。这里就讲一下围绕Channel的组件吧。

如下图，每一个Channel都有自己所属的EventLoop和Pipeline。

![quicker_073d9a21-6713-41fd-a33f-f6de019ee2c7.png](https://i.loli.net/2020/08/15/S6G7NxbHdVBIL5f.png)

pipeline维护了链表形式的ChannelHandlerContext，每次addLast()添加ChannelHandler到pipeline中时，都会被转成ChannelHandlerContext并添加一个链表节点

![quicker_b7df4071-be2d-4ef0-a09a-2b3faec380a2.png](https://i.loli.net/2020/08/15/um8wyI9JDtenTCK.png)

![quicker_598f9744-1bc2-4099-9ab0-a37ba006bf80.png](https://i.loli.net/2020/08/15/6RGKyZInjXhE1Ap.png)

## register注册

Channel初始化完成之后，会执行到注册阶段，如下代码，它提交了一个任务给到EventLoop，上面的分析我们知道，提交EventLoop.execute后，会先将task入队，然后启动死循环执行task，这里是register0方法。

![quicker_710c1d40-8664-4a9f-bd95-866c2d346924.png](https://i.loli.net/2020/08/15/gWciRnxfQ7UetwO.png)

那么register0就是异步通过EventLoop执行的了。main线程继续执行，我们断点在了NioEventLoop的线程，切换过去可以发现它启动了死循环。

![quicker_b07b256d-a2a3-47c6-904b-1a908e0aff2b.png](https://i.loli.net/2020/08/15/gTlKBMfL5HhDONz.png)

![quicker_10eae2ca-d9c1-42cb-95fb-cec12078cfe2.png](https://i.loli.net/2020/08/15/1M7bjCvfzYZ6uhe.png)

调到底层进行注册了。

![quicker_73d60f55-cdf6-45ec-a4d3-b9458ed64304.png](https://i.loli.net/2020/08/15/VvJjShdozryQaI7.png)

接下来看到没有，它要执行ChannelInitilizer的initChannel方法了。

![quicker_bc1378d2-ea10-4911-b037-c9207bb7686d.png](https://i.loli.net/2020/08/15/L4ewQZMd7aDjh9m.png)

它是怎么执行的呢，如下图，调用的是pipeline的execute方法。

![quicker_026b4da5-1ee8-47fc-82cd-017e846c58be.png](https://i.loli.net/2020/08/15/EwfbrJxaD3AnqgS.png)

跟踪进去，开始执行方法了。

![quicker_d57299d1-a4dc-4d53-92ff-531958ff392e.png](https://i.loli.net/2020/08/15/dC8xrWiBqSUXJpZ.png)

跳到我们最初的initChannel方法。可以看到最后提交了一个task到EventLoop中。往pipeline中添加了一个ServerBootstrapAcceptor。bossGroup是怎么实现将连接建立后分配给workGroup的呢，这就是关键。

![quicker_b52a035d-0f7b-48e9-a55a-83868de229c4.png](https://i.loli.net/2020/08/15/XM17cbTqSDVtaQp.png)

## SocketChannel连接分配

我们知道，如果有客户端连接到服务端的话，会依次执行pipeline中的ChannelHandler，上面我们可以看到，NioServerSocketChannle最后一个ChannelHandler就是ServerBootstrapAcceptor连接分配器。

我们用客户端连接上来，发现它执行到了ServerBootstrapAcceptor的channelRead方法。并拿到一个SockerChannel，向SocketChannel添加了ChannelInitializer。然后向childGroup中注册该SocketChannel。

![quicker_b302e290-7fa2-4682-afb5-e14119f2d97e.png](https://i.loli.net/2020/08/15/ZWBJflqR9SnxArs.png)

ChildGroup会选择一个EventLoop来注册这个SocketChannel。

![quicker_83e35bb4-5a5e-45ff-94ca-4b0f5ba0171e.png](https://i.loli.net/2020/08/15/68y2zlTajb1ptD9.png)

至此，Netty服务端启动流程结束。