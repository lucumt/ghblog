---
title: "一次由wait造成的死锁问题分析"
date: 2020-09-12T18:07:27+08:00
lastmod: 2020-09-12T18:07:27+08:00
draft: false
keywords: ["java","multiple-threads","deadlock"]
description: "记录java多线程编程中一次由wait造成的死锁问题分析，给自己和相关人员提供总结与参考"
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

在[多种方式实现在Java中用线程轮流打印ABC](/post/java-concurrency/print-a-b-c-in-turn-by-threads/)中自己本想用单个锁通过调用`wait()`实现轮流打印，但一直遇到全部等待并停止执行的死锁问题，经过网上求助和自己分析，最终找到问题原因，简单记录下。

<!--more-->

## 原始代码

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

## 执行结果&状态查看

运行上述程序，可发现在输出若干字符后，程序处于阻塞状态，输出停止：

![程序停止输出](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads-caused-by-wait/all-threads-stopped-print-characters.png "程序停止输出") 

利用`jps`和`jstack`命令查看，可发现3个进程都处于`WAITING`状态形成类似死锁的现象，由此导致程序停止输出。

![线程都处于等待状态](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads-caused-by-wait/all-threads-are-waiting-for-awake.png "线程都处于等待状态") 

由于多次运行上述代码，均能出现此问题(只不过出现的时间间隔不同)，可知程序代码肯定编写不正确。 

## 原因分析

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

![所有线程均调用wait方法](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads-caused-by-wait/all-threads-invoke-wait-method.png "所有线程均调用wait方法") 

从上图可以看出中无法找出具体的问题复现规律，查看`wait()`的[官方文档](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#wait--)，有如下说明

> The current thread must own this object's monitor. The thread releases ownership of this monitor and waits until another thread notifies threads waiting on this object's monitor to wake up either through a call to the `notify` method or the `notifyAll` method. The thread then waits until it can re-obtain ownership of the monitor and resumes execution.

查看`notifyAll()`的[官方文档](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#notifyAll--)，有如下说明：

> The awakened threads will not be able to proceed until the current thread relinquishes the lock on this object. The awakened threads will compete in the usual manner with any other threads that might be actively competing to synchronize on this object;

从上述说明中可获得如下信息：

1. 调用`wait()`方法后，程序会处于等待状态，同时释放锁
2. 被`notifyAll()`唤醒的线程并不会立即执行，需要等当前调用`notifyAll()`的线程释放对应的锁
3. 被`notfiyAll()`唤醒的线程需要与其它正在执行的线程一并去竞争获取锁，并没有区别对待

至此，可以大致找出问题原因

> 三个线程在某个时间段先后调用了`wait()`方法都进入了`WAITING`状态，同时没有其它线程来唤醒它们，导致最终程序停止输出。

以线程A造成死锁为例，其过程如下:

1. 线程A调用`wait()`方法进入`WAITING`状态并释放锁
2. 线程B获得锁，进入`synchronized`代码块
3. 线程B执行`wait()`方法进行`WAITING`状态并释放锁
4. 线程C获得锁，进入`synchronized`代码块
5. 线程C执行`wait()`方法进行`WAITING`状态并释放锁
6. 至此线程A、线程B、线程C均处于`WAITING`状态，程序停止输出

图示说明如下：

![分析图示1](/blog_img/java-concurrency/analysis-deadlock-in-multiple-threads-caused-by-wait/multi-thread-deadlock-analysis.png "分析图示") 

## 改进代码

找到问题原因后，要修复此问题也很容，只需要避免所有的线程同时调用`wait()`方法即可，可在调用`wait()`前加个判断，确保始终有一个线程不用进入`WAITING`状态，能够释放锁。

### 消除死循环

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

### 实现轮流打印

```java
public class ThreadPrint4Test {

    public static void main(String[] args) {
        new ThreadPrint4Test().testPrint();
    }

    private volatile int count = 0;

    public void testPrint() {
        Object lock = new Object();
        new Thread(new PrintThread("A", lock), "thread-A").start();
        new Thread(new PrintThread("B", lock), "thread-B").start();
        new Thread(new PrintThread("C", lock), "thread-C").start();
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
                        boolean process = "A".equals(value) && count % 3 == 0;
                        if (process) {
                            count = 0;
                        }
                        process = process || ("B".equals(value) && count % 3 == 1);
                        process = process || ("C".equals(value) && count % 3 == 2);
                        if (process) {
                            lock.wait();
                            System.out.println(value);
                            count++;
                            Thread.sleep(500);
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

