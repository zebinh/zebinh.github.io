---
layout: post
title: sleep和wait区别以及while死循环
date: 2019-11-23
tags: Java
---
# sleep和wait区别以及while死循环

## sleep()和wait()区别

sleep()和wait()的区别属于老生常谈了，大部分Java面试或者笔试都会问到。标准的答案是：
+ 线程阻塞，两者都会释放cpu资源
+ sleep()不会释放锁资源，wait()会释放自身持有的锁
+ wait()需要再synchronized中使用，而sleep()不需要

其上第一点，线程阻塞两者都会释放cpu资源，这一点很重要。想想以下执行的代码会有区别吗？
```java
// 代码段1
while (true) {
    System.out.println("Hello World");
}

// 代码段2
while (true) {
    System.out.println("Hello World");
    Thread.sleep(1);
}
```
上面的代码段1，是一个死循环，执行该代码电脑风扇就呼呼响了，cpu占用也提升。而代码段二，也是使用了while(true)包裹，但cpu占用却不会明显提升。

## while死循环

从上面我们知道，如果while(true)中使用了能使得线程阻塞的代码，那么程序将一直运行但cpu不会高占用。JDK源码中大量使用了这个设计。比如，延迟线程池```ScheduledThreadPoolExecutor```，其使用了DelayedWorkQueue队列，将task按执行时间排序，排在队头的第一个执行。

同时，使用for(;;)死循环，比较目标时间和当前时间的毫秒差值delay，如果delay小于等于0则执行该task，否则awaitNanos(delay)阻塞线程。这样就避免了for(;;)占用cpu占满cpu。源码如下：
```java
public RunnableScheduledFuture<?> take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            RunnableScheduledFuture<?> first = queue[0];
            if (first == null)
                available.await();
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    return finishPoll(first);
                first = null; // don't retain ref while waiting
                if (leader != null)
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && queue[0] != null)
            available.signal();
        lock.unlock();
    }
}
```

## synchronized原理

synchronized关键字我们再熟悉不过了，其作用有两点：
+ 线程同步
+ 线程协作

使用synchronized同步块或者同步方法时，只有一个线程能进入该区域，叫做线程同步。synchronized配置wait()和notify()使用，可以在两个或多个线程之间协调资源，叫做线程协作，这也是为什么wait()和notify()需要在synchronized中使用的原因。

那synchronized的原理是什么呢？--监视器monitor

synchronized后一般带有锁对象，即synchronized(this)或者默认的锁对象。synchronized底层由Java对象头和monitor监视器实现。

监视器指令有ACC_SYNCHRONIZED和monitorenter、monitorexit。

而每个Object对象在对象头都有指向ObjectMonitor数据结构的指针，如下图，ObjectMonitor中包含owner保存当前执行的线程，EntryList保存执行到synchronized的线程，waitSet保存执行了wait()方法的线程，线程是以ObjectWaiter形式封装保存到队列的，count表示重入次数，因此synchronized是可重入锁。

![20191123133707.png](https://i.loli.net/2019/11/23/XqwudQrcFKtVAOs.png)

总结以上的内容，就是synchronized围绕对象锁，对象锁维护了EntryList用于线程同步，WaitSet用于线程协作。

## JUC中的ReentrantLock

synchronized用起来比较繁琐难用。别担心，JDK给我们提供了JUC包中的ReentrantLock，是synchronized的完美代替品。具体的内容可以看相关资料，这里不展开讨论。

## volatile关键字

其实多线程的问题，涉及到了JMM内存模式，多线程会发生以下几个问题：
+ 原子性
+ 有序性
+ 可见性

解决了上面的三个问题，你的程序就是线程安全的了。而volatile可以保证有序性和可见性。

volatile底层由内存屏障实现，内存屏障是cpu的指令，其禁止在内存屏障前后的指令中插入其他指令，保证了有序性，同时内存屏障强制刷缓存，保证了可见性。单例的DCL(Double Check Lock)是典型的使用volatile的场景，因为new一个实例需要三步：分配内存、初始化、指针指向，这三步不能保证有序性，可以使用volatile解决。


## JUC包

JUC包简直是多线程开发的神器，JUC主要的概念是CAS,AQS，基本所有类都是构建在其上。JUC包基本可以说是用来替代volatile和synchronized关键字的，其中atomic包用来替代volatile，lock包用来替代synchronized，lock中的condition实现了await(),signal()用来替代synchronized的wait(),notify。JUC包中的BlockingQueue是使用Condition来实现的。线程池则是构建在BlockingQueue队列上。