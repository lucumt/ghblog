---
title: "[译]如何处理InterruptedException"
date: 2018-12-15T11:57:36+08:00
lastmod: 2018-12-15T11:57:36+08:00
draft: false
keywords: ["Java多线程"]
description: "翻译外网的一篇文章，主要是关于在Java多线程编程中如何处理InterruptedException"
autoCollapseToc: true
tags: ["Java"]
categories: ["Java编程","Java多线程","翻译"]
author: "Rosen Lu"
---

本文翻译自 **[Dealing with InterruptedException](https://www.ibm.com/developerworks/library/j-jtp05236/)** 

这个故事可能很熟悉: 你正在编写一个测试程序，需要将程序暂停一段时间,于是你调用了`Thread.sleep()`来实现。
但此时编译器或IDE会立即提示你没有处理非运行时异常 **[InterruptedException](https://docs.oracle.com/javase/8/docs/api/?java/lang/InterruptedException.html)**。那么，什么是`InterruptedException`,为什么我们要必须处理它呢？

<!--more-->

最常见的处理方式是将该异常吞没掉: 通过`try-catch`捕获该异常然后不进行任何处理(或许会用日志来记录异常，但并没有改进多少)，
我们将在代码清单4中看见此方式的使用。不幸的是，这种方式丢弃了有关中断产生的重要信息，而这可能会影响应用程序及时取消执行或关闭的能力。

## 阻塞方法

当一个方法抛出InterruptedException异常时，除了该异常之外它还会告诉你额外的一些信息。如果你问的好，它会告诉你这是一个阻塞方法，
它会尝试解除阻塞并且提前返回。

阻塞方法不同于一个需要长时间返回的普通方法，普通方法的完成仅取决于你要求它做多少工作以及是否有足够的计算资源(CPU循环和内存)，另一方面，阻塞方法的完成度还取决于某些外部事件，例如计时器到期、I/O操作完成或另一个线程的操作(释放锁、设置标志位或将任务放到一个工作队列中等)。普通的方法只要其工作任务完成了该方法就完成，而阻塞方法由于它们依赖于外部事件而不太可预测。由于很难预测何时完成，阻塞方法会影响响应能力。

由于阻塞方法可能由于等待的事件永远不发生而一直阻塞，所以通常能取消该方法对阻塞方法很有用(对于长时间运行的非阻塞方法，通常也是有用的)。可取消的操作是指能从外部提前结束一个需要它自己完成的任务。`Object.wait()`和Thread提供的`Thread.sleep()`即是一种取消机制，它允许一个线程请求另外一个线程提前停止它正在执行的任务。当一个方法抛出`InterruptedException`异常时，意味着如果执行该方法的线程被中断，它将通过抛出`InterruptedException`异常来尝试停止当前任务并且提前返回。一个设计良好的阻塞库方法应该及时响应中断并抛出`InterruptedException`异常，因而可以在可取消的任务中使用而不影响响应能力。

### 线程中断

每个线程都一个与之关联的布尔属性来表示其中断状态。中断状态的初始值为false，当一个线程被其它线程调用`Thread.interrupt()`方法中断时，会根据实际情况做出响应: 如果该线程正在执行低级别的可中断方法(如`Thread.sleep()`、`Thread.join()`或`Object.wait()`)，则会解除阻塞并抛出`InterruptedException`异常，否则`Thread.interrupt()`仅设置线程的中断状态，在该被中断的线程中稍后可通过轮询中断状态来决定是否要停止当前正在执行的任务。中断状态可通过`Thread.isInterrupted()`来读取，也可通过不规范命名的`Thread.interrupted()`在单个操作中读取和清除中断状态。

中断是一种协作机制。当一个线程中断了另外一个线程时，被中断的线程没必要停止正在执行的任务，相反，中断只是礼貌的要求被中断的线程在它方便的时候停止它当前正在执行的任务。有些方法如`Thread.sleep()`会认真对待这个要求，而其它一些方法则不需要注意中断。不阻塞但需要长时间执行的方法也可通过轮询中断状态来提前返回，你也可以忽略中断求，但这样做可能会影响响应性。

中断的协作特性的一个好处是它为安全的构建可取消的任务提供了更大的灵活性。实际上我们很少想要立即终止一个任务，如果任务在更新期间被取消，程序数据结构可能会处于不一致的状态。而中断操作允许我们在任务终止之前做其它的一些操作，如清理任何正在运行的任务、恢复不变量、将终端请求通知给其它任务等。

## InterruptedException

如果抛出`InterruptedException`异常意味着该方法是一个阻塞方法，那么调用一个阻塞方法意味着调用方法也是阻塞方法，我们应该有一个处理`InterruptedException`异常的策略。通过最简单的策略是直接抛出`InterruptedException`，如代码清单1中puskTask()和getTask()方法所示,这样做即可使方法响应中断，并且通常只需要在throws字句中添加`InterruptedException`。

代码清单1： 通过向调用者传递`InterruptedException`来避免捕获
```java
public class TaskQueue {
    private static final int MAX_TASKS = 1000;
 
    private BlockingQueue<Task> queue 
            = new LinkedBlockingQueue<Task>(MAX_TASKS);
 
    public void putTask(Task r) throws InterruptedException { 
        queue.put(r);
    }
 
    public Task getTask() throws InterruptedException { 
        return queue.take();
    }
}
```

有时在传播异常之前需要进行一些清理，在这种情况下，你可以捕获`InterruptedException`，执行清理然后重新抛出异常。代码清单2，一种用于匹配在线游戏服务中玩家的机制展示了这种技术。`matchPlayers()`方法等待两个玩家到达后就开始一个新的游戏，如果在一个玩家到达后但在第二个玩家到达之前被中断，它会在重新抛出`InterruptedException`异常之前将该玩家放回队列，这样玩家的请求就不会丢失。

代码清单2: 在重新抛出`InterruptedException`之前基于任务进行特定清理
```java
public class PlayerMatcher {
    private PlayerSource players;
 
    public PlayerMatcher(PlayerSource players) { 
        this.players = players; 
    }
 
    public void matchPlayers() throws InterruptedException { 
        Player playerOne, playerTwo;
         try {
             while (true) {
                 playerOne = playerTwo = null;
                 // 等两个玩家到达后开始一个新的游戏
                 playerOne = players.waitForPlayer(); // 可能抛出InterruptedException
                 playerTwo = players.waitForPlayer(); // 可能抛出InterruptedException
                 startNewGame(playerOne, playerTwo);
             }
         }
         catch (InterruptedException e) {  
             //如果一个玩家达到后线程被中断，将这个玩家放回队列
             if (playerOne != null)
                 players.addFirst(playerOne);
             //重新抛出异常
             throw e;
         }
    }
}
```

### 不要吞没中断

有些时候抛出`InterruptedException`异常并不合适，例如在由`Runnable`定义的任务中调用一个可中断方法，在这种情形下，你不能抛出`InterruptedException`，但你也不想做其它任何事情。当阻塞方法检测到中断并抛出`InterruptedException`时，它会清除中断状态。如果捕获`InterruptedException`异常但无法重新抛出它，则应保留中断发生时的证据，以便调用堆栈上的代码可以了解中断并在必要时对其进行响应。如代码清单3所示，这个任务是通过调用`interrupt()`来 *重新中断(reinterrupt)* 当前线程完成的。每当你捕获`InterruptedException`并且不重新抛出它时，要在返回之前重新中断当前线程(即清除中断状态)。

代码清单3: 捕获`InterruptedException`后恢复中断状态
```
public class TaskRunner implements Runnable {
    private BlockingQueue<Task> queue;
 
    public TaskRunner(BlockingQueue<Task> queue) { 
        this.queue = queue; 
    }
 
    public void run() { 
        try {
             while (true) {
                 Task task = queue.take(10, TimeUnit.SECONDS);
                 task.execute();
             }
         }
         catch (InterruptedException e) { 
             //重新设置当前线程的中断状态
             Thread.currentThread().interrupt();
         }
    }
}
```

对于`InterruptedException`最糟糕的处理是吞没它，在捕获它之后既不重新抛出它也不重新设置当前线程的中断状态。

代码清单4: 吞下中断--不要这样做
```java
// 不要这么做
public class TaskRunner implements Runnable {
    private BlockingQueue<Task> queue;
 
    public TaskRunner(BlockingQueue<Task> queue) { 
        this.queue = queue; 
    }
 
    public void run() { 
        try {
             while (true) {
                 Task task = queue.take(10, TimeUnit.SECONDS);
                 task.execute();
             }
         }
         catch (InterruptedException swallowed) { 
             /* 不要这么做 - 相反要恢复线程的中断状态 */
         }
    }
}
```

如果你无法重新抛出`InterruptedException`，不论你是否计划对中断请求进行响应，你仍然应该重新设置当前线程的中断状态，因为单个中断请求可能有多个接收人。标准线程池(`ThreadPoolExecutor`)中的工作线程实现了响应中断，因此中断线程池中运行的任务可能会同时影响到取消任务和通知执行线程当前线程池正在关闭。如果该任务吞下中断请求，则工作线程可能不会知道该中断被请求过，这可能会延迟应用程序或关闭服务。

## 实现可取消的任务

在Java语言规范中没有任何内容可以中断任何特定语义，但在较大程序中，很难保持除了取消操作之外的任何中断语义。依赖于具体的业务活动，用户可以通过GUI或JMS、WebService服务等来请求取消操作。也可以由程序逻辑请求实现，例如，网络爬虫在检测到磁盘空间已满时自动关闭，或者并行算法可能会启动多个线程来搜索解决方案的不同区域并在其中一个找到解决方案后取消其余的。

一个任务不能仅仅因为它可以取消而意味着它需要立即响应中断请求。对于通过代码循环执行的任务，通常是在每个循环中检查中断状态。根据循环执行的时间长短，在任务代码通知线程中断之前可能需要一些时间(通过调用`Thread.isInterrupted()`轮询中断中断或调用一个阻塞方法)。如果任务需要更具响应性，则可以更频繁的轮询中断状态。阻塞方法通常在进入该方法时立即轮询中断状态，如果要提高响应性，可以直接抛出`InterruptedException`异常。

当你知道一个线程即将结束执行时，吞下一个中断是可接受的。只有当调用可中断方法的类是`Thread`的一部分而不是`Runnable`或通用库代码时才会出现此种情形，如代码清单5所示。它创建了一个枚举质数的线程，并且允许线程在中断时退出。这个寻找质数的循环在两个地方检查中断：一个通过轮迅while循环判断条件中的`isInterrupted()`方法，另一个则是通过调用阻塞的`BlockingQueue.put()`方法。

代码清单5: 在知道线程将结束时可以吞下中断
```java
public class PrimeProducer extends Thread {
    private final BlockingQueue<BigInteger> queue;
 
    PrimeProducer(BlockingQueue<BigInteger> queue) {
        this.queue = queue;
    }
 
    public void run() {
        try {
            BigInteger p = BigInteger.ONE;
            while (!Thread.currentThread().isInterrupted())
                queue.put(p = p.nextProbablePrime());
        } catch (InterruptedException consumed) {
            /* 允许线程退出 */
        }
    }
 
    public void cancel() { interrupt(); }
}
```

### 非中断阻塞

并非所有的阻塞方法都会抛出`InterruptedException`。输入流和输出流由于等待I/O操作完成而阻塞，但它们不会抛出`InterruptedException`异常，并且如果它们被中断，也不会提前返回。但在套接字I/O的情形下，如果一个线程关闭套接字，在其它线程中的阻塞I/O操作将会通过`SocketException`提早结束。java.nio包中的非阻塞I/O操作也不支持可中断的I/O操作，但可以通过关闭通道或请求唤醒`Selector`来实现类似的阻塞操作。类似的，尝试获取内部锁(进入一个同步块)不能被中断，但使用`ReentrantLock`能实现可中断的获取锁。

### 非取消任务

有些任务只是简单的拒绝被中断从而导致它们无法取消。但即使是不可取消的任务也应该保留中断状态，以防在调用堆栈上的代码想要在不可取消的任务完成后响应中断。代码清单6显示了一个方法等待阻塞队列直到其中某个元素可用，无论它是否被中断。为了安分守己，该方法在完成后会清除最终代码块中的中断状态，以避免剥夺调用者发出的中断请求。由于会导致无限循环而不能在该方法中恢复中断状态。(`BlockingQueue.take()`可以在调用时立即轮询中断状态，如果发现有中断状态集则立即抛出`InterruptedException`异常。)

代码清单6: 非取消的任务在返回之前恢复中断状态
```java
public Task getNextTask(BlockingQueue<Task> queue) {
    boolean interrupted = false;
    try {
        while (true) {
            try {
                return queue.take();
            } catch (InterruptedException e) {
                interrupted = true;
                // 虽然发生中断，但是继续尝试
            }
        }
    } finally {
        if (interrupted)
            Thread.currentThread().interrupt();
    }
}
```

## 总结

你可以使用Java平台提供的协作中断机制来构建灵活的取消策略。线程活动可以决定它们是否可以取消、如何响应中断，如果立即返回会影响应用程序的完整性时，它们可以推迟中断以执行特定于任务的清理活动。即是想完全忽略代码中的中断，也要确保在捕获`InterruptedException`时恢复中断状态，并且不要重新抛出它以便调用它的代码不会被剥夺对于中断发生的感知。


<–翻译结束!–>