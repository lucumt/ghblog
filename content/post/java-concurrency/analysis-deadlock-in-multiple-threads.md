---
title: "一次Java多线程中的类死锁问题分析"
date: 2020-09-12T18:07:27+08:00
lastmod: 2020-09-12T18:07:27+08:00
draft: true
keywords: ["java","multiple-threads","deadlock"]
description: "一次Java多线程中的类似死锁问题分析"
tags: ["Java","Java Concurrency"]
categories: ["Java编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
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

在[多种方式实现在Java中用线程轮流打印ABC](/post/java-concurrency/print-a-b-c-in-turn-by-threads/)中自己本想用单个锁实现轮流打印，但一直遇到类似死锁的全部等待并停止执行的问题，经过网上求助和自己分析，最终找到问题原因，简单记录下。

<!--more-->

# 原始代码

```java
public class ThreadPrint4Test {

    public static void main(String[] args) {
        new ThreadPrint4Test().testPrint();
    }

    public void testPrint() {
        Object lock = new Object();
        new Thread(new PrintThread("A",lock),"thread-A").start();
        new Thread(new PrintThread("B",lock),"thread-B").start();
        new Thread(new PrintThread("C",lock),"thread-C").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        new Thread(() -> {
            synchronized (lock) {
                lock.notifyAll();
            }
        }).start();
    }

    class PrintThread implements Runnable {

        private Object lock;
        private String value;

        public PrintThread(String value, Object lock) {
            this.value = value;
            this.lock = lock;
        }

        public void run() {
            while (true) {
                try {
                    synchronized (lock) {
                        lock.wait();
                        System.out.println(LocalTime.now() + "\t" + value);
                        lock.notifyAll();
                    }
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}
```

# 执行结果&状态查看

运行上述程序，可发现在输出若干字符后，程序处于阻塞状态，输出停止：

![程序停止输出](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads/all-threads-stopped-print-characters.png "程序停止输出") 

利用`jps`和`jstack`命令查看，可发现3个进程都处于`WAITING`状态形成类似死锁的现象，由此导致程序停止输出。

![线程都处于等待状态](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads/all-threads-are-waiting-for-awake.png "线程都处于等待状态") 

由于多次运行上述代码，均能出现此问题(只不过出现的时间间隔不同)，可知程序代码肯定编写不正确。 

# 原因分析

最开始检查代码时看起来一切正常:

1. 某线程首先通过`wait()`进入等待状态
2. 只有被唤醒后才能继续往下执行输出代码
3. 输出完毕后通过`notifyAll()`
4. 被`notifyAll()`唤醒的线程从步骤2开始往下执行
5. 之后该线程调用`wait()`进入等待状态，重复步骤1实现不间断的打印

```java
public void run() {
    while (true) {
        try {
            synchronized (lock) {
                lock.wait();
                System.out.println(LocalTime.now() + "\t" + value);
                lock.notifyAll();
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

上述推理与代码看起来很完美，无懈可击，但程序执行时就是每次都不能一直打印下去，肯定是自己的理解有问题。

为了便于定位问题，将`run()`方法修改为如下代码：

```java
public void run() {
    while (true) {
        try {
            synchronized (lock) {
                String name = Thread.currentThread().getName();
                System.out.println(name + "\t" + LocalTime.now() + "\t" + " will invoke wait");
                lock.wait();
                System.out.println(name + "\t" + LocalTime.now() + "\t" + " is notified");
                System.out.println(name + "\t" + LocalTime.now() + "\t" + value);
                lock.notifyAll();
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
  }
}
```

之后多次重新执行该程序，执行结果类似如下：

![所有线程均调用wait方法](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads/all-threads-invoke-wait-method.png "所有线程均调用wait方法") 

从上图可以看出当3个线程全部进入`WAITING`状态后的一个共同点：线程在执行打印输出后，又率先调用了`wait()`方法。

至此，造成3个线程停止打印输出的原因已经明确：

>  某个线程在执行打印输出后，通过`notifyAll()`方法**唤醒的是线程自身**，然后该线程又调用`wait()`方法导致所有线程都处于`WAITING`状态，从而停止打印！

对于同一把锁而言，难道`notifyAll()`唤醒的不是已经调用`wait()`方法的线程么，为什么还能唤醒线程本身呢，查看其[官方文档](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#notifyAll--)，有如下说明：

> Wakes up all threads that are waiting on this object's monitor. A thread waits on an object's monitor by calling one of the `wait` methods.
>
> <br>
>
> The awakened threads will not be able to proceed until the current thread relinquishes the lock on this object. The awakened threads will compete in the usual manner with any other threads that might be actively competing to synchronize on this object; for example, the awakened threads enjoy no reliable privilege or disadvantage in being the next thread to lock this object.