---
title: "一次由notifyAll造成的死锁问题分析"
date: 2020-09-12T18:07:27+08:00
lastmod: 2020-09-12T18:07:27+08:00
draft: true
keywords: ["java","multiple-threads","deadlock"]
description: "一次由notifyAll造成的死锁问题分析"
tags: ["Java","Java Concurrency"]
categories: ["Java编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
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

在[多种方式实现在Java中用线程轮流打印ABC](/post/java-concurrency/print-a-b-c-in-turn-by-threads/)中自己本想用单个锁通过调用`notifyAll()`实现轮流打印，但一直遇到全部等待并停止执行的死锁问题，经过网上求助和自己分析，最终找到问题原因，简单记录下。

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

![程序停止输出](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads-caused-by-notifyall/all-threads-stopped-print-characters.png "程序停止输出") 

利用`jps`和`jstack`命令查看，可发现3个进程都处于`WAITING`状态形成类似死锁的现象，由此导致程序停止输出。

analysis-deadlock-in-multiple-threads-caused-by-notifyall![线程都处于等待状态](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads/all-threads-are-waiting-for-awake.png "线程都处于等待状态") 

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

![所有线程均调用wait方法](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads-caused-by-notifyall/all-threads-invoke-wait-method.png "所有线程均调用wait方法") 

从上图可以看出当3个线程全部进入`WAITING`状态后的一个共同点：线程在执行打印输出后，又率先调用了`wait()`方法。

至此，造成3个线程停止打印输出的原因已经明确：

>  某个线程在执行打印输出后，通过`notifyAll()`方法**唤醒的是线程自身**，然后该线程又调用`wait()`方法导致所有线程都处于`WAITING`状态，从而停止打印！

对于同一把锁而言，难道`notifyAll()`唤醒的不是已经调用`wait()`方法的线程么，为什么还能唤醒线程本身呢，查看其[官方文档](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#notifyAll--)，有如下说明：

> The awakened threads will not be able to proceed until the current thread relinquishes the lock on this object. The awakened threads will compete in the usual manner with any other threads that might be actively competing to synchronize on this object;

从上述说明中可获得如下信息：

1. 被`notifyAll()`唤醒的线程并不会立即执行，需要等当前调用`notifyAll()`的线程释放对应的锁
2. 被`notfiyAll()`唤醒的线程需要与其它正在执行的线程一并去竞争获取锁，并没有区别对待

至此，造成死锁的原因已经找出:

> 某个线程调用 `notifyAll()`来唤醒其它的线程后，被唤醒的线程不会立即执行而是需要等当前线程释放锁之后才能执行，但是当前线程在释放锁之后通过`while`循环再次由`synchronized`获取锁时**先于其它被`notifyAll()`唤醒的线程获取到了锁**，之后执行`wait()`导致3个线程全部处于等待状态!

以线程A造成死锁为例，其过程如下:

1. 线程A调用`wait()`方法进入`WAITING`状态
2. 其它线程调用`notifyAll()`方法唤醒处于`WAITING`状态的线程，线程A碰巧被唤醒
3. 线程A获得锁，得到执行机会，打印输出字符A
4. 线程A调用`notifyAll()`方法唤醒其它全部处于`WAITING`状态的线程
5. 在线程A没有释放锁之前，线程B和线程C均处于`BLOCKED`状态
6. 线程A释放锁，接着通过`while`再次进入循环
7. 线程A通过`synchronzied`与其它线程处于`BLOCKED`状态的线程竞争锁[^1]
8. 线程A竞争锁成功，调用`wait()`方法处于`WAITING`状态并释放锁
9. 线程B获得锁，进入`synchronized`同步块
10. 线程B调用`wait()`方法并释放锁，线程B处于`WAITING`状态
11. 线程C获得锁，进入`synchronized` 同步块
12. 线程C调用`wait()`方法并释放锁，线程C处于`WAITING`状态
13. 至此3个线程都处于`WAITING`状态，产生死锁，程序停止输出

图示说明如下：

* 线程A在打印输出后通过`while`和`synchronized`再次获取锁，线程B和线程C依旧处于`BLOCKED`状态

  ![分析图示1](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads-caused-by-notifyall/multi-thread-deadlock-analysis-1.png "分析图示1") 

* ce

* ce

* ce

# 改进代码

找到问题原因后，要修复此问题也很容，只需要在调用`wait()`时判断下前一次执行打印输出的线程和当前线程是否为同一个线程，若为同一个线程，则直接跳过`wait()`调用即可(`notifyAll()`方法需要保留，确保能正常唤醒其它线程)

## 消除死循环

```java
public class ThreadPrint4Test {

    public static void main(String[] args) {
        new ThreadPrint4Test().testPrint();
    }

    private volatile String runThread;

    public void testPrint() {
        Object lock = new Object();
        new Thread(new PrintThread("A", lock), "thread-A").start();
        new Thread(new PrintThread("B", lock), "thread-B").start();
        new Thread(new PrintThread("C", lock), "thread-C").start();
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
                        boolean process = value.equals(runThread);
                        if (!process) {
                            lock.wait();
                            System.out.println(LocalTime.now() + "\t" + value);
                            runThread = value;
                        }
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

## 实现轮流打印



[^1]: 基于`Java` 中多线程的执行机制，此种情况是可能发生的

