---
title: "多种方式实现在Java中用线程轮流打印ABC"
date: 2020-09-08T11:11:06+08:00
lastmod: 2020-09-08T11:11:06+08:00
draft: false
keywords: ["java","thread"]
description: "由于面试时踩过坑，记录下多种方式实现在Java中用线程轮流打印ABC的实现"
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

在`Java`中采用3个线程不停的轮流打印ABC这3个字符，采用不同的方式实现。

<!--more-->

## 原理分析

实现思路如下：

* 要在一个`while`循环中，实现不停的打印
* 多个线程间需要依次唤醒下一个线程，实现依次打印，此时会形成死循环
* 为了打破死循环，还需要一个启动线程延后调用

![原理图](/blog_img/java-concurrency/print-a-b-c-in-turn-by-threads/threads-print-character-process.png "原理图") 

## 代码实现

### 基于wait&notify

```java
public class ThreadPrint1Test {

    public static void main(String[] args) {
        new ThreadPrint1Test().testPrint();
    }

    public void testPrint() {
        Object lockA = new Object();
        Object lockB = new Object();
        Object lockC = new Object();

        new PrintThread("A", lockA, lockB).start();
        new PrintThread("B", lockB, lockC).start();
        new PrintThread("C", lockC, lockA).start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        new Thread(() -> {
            synchronized (lockA) {
                lockA.notify();
            }
        }).start();
    }

    class PrintThread extends Thread {

        private Object waitLock;
        private Object notifyLock;
        private String value;

        public PrintThread(String value, Object waitLock, Object notifyLock) {
            this.value = value;
            this.waitLock = waitLock;
            this.notifyLock = notifyLock;
        }

        public void run() {
            while (true) {
                synchronized (waitLock) {
                    try {
                        waitLock.wait();
                        Thread.sleep(500);
                        System.out.println(value);
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                    synchronized (notifyLock) {
                        notifyLock.notify();
                    }
                }
            }
        }
    }
}
```

### 基于ReentrantLock&Condition

```java
public class ThreadPrint2Test {

    public static void main(String[] args) {
        new ThreadPrint2Test().testPrint();
    }

    public void testPrint() {
        Lock lock = new ReentrantLock();
        Condition conditionA = lock.newCondition();
        Condition conditionB = lock.newCondition();
        Condition conditionC = lock.newCondition();

        new Thread(new PrintThread("A", lock, conditionA, conditionB), "thread-a").start();
        new Thread(new PrintThread("B", lock, conditionB, conditionC), "thread-b").start();
        new Thread(new PrintThread("C", lock, conditionC, conditionA), "thread-c").start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        new Thread(() -> {
            lock.lock();
            conditionA.signal();
            lock.unlock();
        }, "init").start();
    }

    class PrintThread implements Runnable {

        private Lock lock;
        private Condition ca;
        private Condition cb;
        private String value;

        public PrintThread(String value, Lock lock, Condition ca, Condition cb) {
            this.value = value;
            this.lock = lock;
            this.ca = ca;
            this.cb = cb;
        }

        public void run() {
            while (true) {
                try {
                    lock.lock();
                    ca.await();
                    System.out.println(value);
                    Thread.sleep(500);
                    cb.signal();
                    lock.unlock();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}
```

### 基于Semaphore

```java
public class ThreadPrint3Test {

    public static void main(String[] args) {
        new ThreadPrint3Test().testPrint();
    }

    public void testPrint() {
        Semaphore sa = new Semaphore(1);
        Semaphore sb = new Semaphore(1);
        Semaphore sc = new Semaphore(1);

        try {
            sb.acquire();
            sc.acquire();
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        new PrintThread("A", sa, sb).start();
        new PrintThread("B", sb, sc).start();
        new PrintThread("C", sc, sa).start();
    }

    class PrintThread extends Thread {

        private Semaphore sa;
        private Semaphore sb;
        private String value;

        public PrintThread(String value, Semaphore sa, Semaphore sb) {
            this.value = value;
            this.sa = sa;
            this.sb = sb;
        }

        public void run() {
            while (true) {
                try {
                    sa.acquire();
                    System.out.println(value);
                    Thread.sleep(500);
                    sb.release();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}
```

### 单个锁&volatile变量

此种方式不需要唤醒线程，同时便于很方便的修改线程数目。

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

### 基于LockSupport

此种方式更加简洁

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.LockSupport;

public class ThreadPrintTestA {

    public static void main(String[] args) {
        PThread ta = new PThread("A");
        PThread tb = new PThread("B");
        PThread tc = new PThread("C");

        ta.setWakeupThread(tb);
        tb.setWakeupThread(tc);
        tc.setWakeupThread(ta);

        ta.setName("thread-1");
        tb.setName("thread-2");
        tc.setName("thread-3");

        ta.start();
        tb.start();
        tc.start();

        try {
            TimeUnit.SECONDS.sleep(1);
            LockSupport.unpark(ta);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}

class PThread extends Thread {

    private String str;
    private Thread wakeupThread;

    public PThread(String str) {
        this.str = str;
    }

    public void setWakeupThread(Thread wakeupThread) {
        this.wakeupThread = wakeupThread;
    }

    @Override
    public void run() {
        try {
            while (true) {
                LockSupport.park();
                System.out.println(str + "\t" + currentThread().getName());
                TimeUnit.MILLISECONDS.sleep(100);
                LockSupport.unpark(wakeupThread);
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

