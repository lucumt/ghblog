---
title: "Java中wait和timed wait方法的区别与使用场景"
date: 2020-08-06T09:28:43+08:00
lastmod: 2020-08-06T09:28:43+08:00
draft: true
keywords: ["Java多线程","wait","wait time"]
description: "Java中wait和timed wait方法的区别与使用场景"
tags: ["Java","Java Concurrency"]
categories: ["Java编程","Java多线程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""
---

简要对比`Java`中[wait()](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#wait--)和[wait(long timeout)](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#wait-long-)的用法与区别。

<!--more-->

从`JDK`的API可知2者均为[Object](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html)类的方法，即`Java`中所有的对象都能调用这2个方法，本文基于`JDK`的官方API简要介绍2者的使用以及异同

## wait方法

在`wait()`方法的官方文档中有如下说明：

> Causes the current thread to wait until another thread invokes the [`notify()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#notify--) method or the [`notifyAll()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#notifyAll--) method for this object. In other words, this method behaves exactly as if it simply performs the call `wait(0)`.
>
> <br>
>
> The current thread must own this object's monitor. The thread releases ownership of this monitor and waits until another thread notifies threads waiting on this object's monitor to wake up either through a call to the `notify` method or the `notifyAll` method. The thread then waits until it can re-obtain ownership of the monitor and resumes execution.

## timed wait方法

在`wait(long timeout)`的官方文档中有如下说明：

> Causes the current thread to wait until either another thread invokes the [`notify()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#notify--) method or the [`notifyAll()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#notifyAll--) method for this object, or a specified amount of time has elapsed.
>
> The current thread must own this object's monitor.
>
> This method causes the current thread (call it T) to place itself in the wait set for this object and then to relinquish any and all synchronization claims on this object. Thread T becomes disabled for thread scheduling purposes and lies dormant until one of four things happens:
>
> - Some other thread invokes the `notify` method for this object and thread T happens to be arbitrarily chosen as the thread to be awakened.
> - Some other thread invokes the `notifyAll` method for this object.
> - Some other thread [interrupts](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#interrupt--) thread T.
> - The specified amount of real time has elapsed, more or less. If `timeout` is zero, however, then real time is not taken into consideration and the thread simply waits until notified.
>
> The thread T is then removed from the wait set for this object and re-enabled for thread scheduling. It then competes in the usual manner with other threads for the right to synchronize on the object; once it has gained control of the object, all its synchronization claims on the object are restored to the status quo ante - that is, to the situation as of the time that the `wait` method was invoked. Thread T then returns from the invocation of the `wait` method. Thus, on return from the `wait` method, the synchronization state of the object and of thread `T` is exactly as it was when the `wait` method was invoked.

## 总结

**相同点**:

* 均为`Object`类的方法，即`Java`中所有的对象都能调用这2个方法
* 调用时均需要先获取调用对象的锁之后才能调用
* 调用后均放弃对于锁的持有，进入等待状态(`WAITING`或`TIMED_WAINT`)
* 均可通过`notfiy()`或`notifyAll()`方法唤醒，被唤醒后需要等待其它线程释放锁或与其它线程竞争锁
* 在调用时均需要捕获[InterruptedException](https://docs.oracle.com/javase/8/docs/api/java/lang/InterruptedException.html)异常，均可通过调用`interrupt()`来强制中断

**不同点**:

* `wait()`调用后的状态为`WAITNG`，`wait(long timeout)`调用后的状态为 `TIMED_WAITING`
* `wait()`调用后只能通过其它线程调用`notify()`或者`notifyAll()`进行唤醒，`wait(long timeout)`除了这2个方法之外，当到达指定的时间还可自行唤醒
* `wait()`等价于`wait(0)`



参考：

1. https://www.oreilly.com/library/view/java-threads-second/1565924185/ch04s04.html