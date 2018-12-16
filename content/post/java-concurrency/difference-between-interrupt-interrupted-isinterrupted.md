---
title: "Java中interrupt()、interrupted()、isInterrupted()的区别"
date: 2018-12-03T22:57:56+08:00
lastmod: 2018-12-03T22:57:56+08:00
draft: true
keywords: ["Java多线程"]
description: "Java中interrupt()、interrupted()、isInterrupted()的区别"
tags: ["Java"]
categories: ["Java编程","Java多线程"]
author: "Rosen Lu"

---
最近复习Java多线程相关知识时，发现线程中断的`interrupt()`、`interrupted()`、`isInterrupted()`这3个方法容易让人产生混淆，结合官网的API以及实际代码验证，先将它们简单记录下。
<!--more-->

## 相关使用说明

下表是根据JDK官网的API总结出的这3个方法的使用说明,从表中可以看出中断线程实际使用的是`interrupt()`，而`interrupted()`和`isInterrupted()`则用来检测线程是否中断，区别为前者会清除中断状态标记。

| &nbsp;&nbsp;方法&nbsp;&nbsp; | 是否静态 |返回值|作用|
| --- | :----: | :----: |---|
| **[interrupt()](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#interrupt--)** | 否 | 无 |中断调用该方法的当前线程|
| **[interrupted()](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#interrupted--)** | 是 | 有 |检测当前线程是否被中断，如已被中断过则清除中断状态|
| **[isInterrupted()](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#isInterrupted--)** | 否 | 有 |检测调用该方法的线程是否被中断，不清除中断标记|

下面这个程序展示通过2个线程展示了这3个方法的用法，首先启动`threadA`，间隔30毫秒后启动`threadB`并在`threadB`中调用`interrupt()`中断`threadA`,然后观察`threadA`的中断状态。
```java
public class InterruptedTest {

	@Test
	public void testThread() {

		Thread threadA = new Thread(() -> {
			while (!Thread.currentThread().isInterrupted()) {
				System.out.println(Thread.currentThread().getName() + "\t" + LocalTime.now());
			}
			System.out.println(Thread.currentThread().isInterrupted());
			System.out.println(Thread.interrupted());
			System.out.println(Thread.currentThread().isInterrupted());
		}, "Thread_A");

		Thread threadB = new Thread(() -> {
			System.out.println(Thread.currentThread().getName() + " interrupt ThreadA");
			threadA.interrupt();
		}, "Thread_B");

		threadA.start();
		try {
			Thread.sleep(30);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		threadB.start();
	}

}
```
根据前述的方法说明，程序的输出结果应改为true、true、false，实际运行结果类似如下,从图中可知其输出结果与理论分析相符。  
![程序运行结果1](/blog_img/difference-between-interrupt-interrupted-isinterrupted/thread_interrupted_running_result_1.png "程序运行结果")  

将上述代码修改如下,首先给`threadA`加上同步锁并调用`wait()`或`sleep()`方法，然后检测其中断状态。  
```java
public class InterruptedTest {

	@Test
	public void testThread() {
		Object lock = new Object();
		Thread threadA = new Thread(() -> {
			synchronized(lock) {
				try {
					Thread.sleep(1000);
					//lock.wait();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(Thread.currentThread().isInterrupted());
				System.out.println(Thread.interrupted());
				System.out.println(Thread.currentThread().isInterrupted());
			}
		}, "Thread_A");
		
		Thread threadB = new Thread(() -> {
			System.out.println(Thread.currentThread().getName() + " will interrupt ThreadA");
			threadA.interrupt();
		}, "Thread_B");
		
		threadA.start();
		try {
			Thread.sleep(30);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		threadB.start();
	}

}
```
分别调用`sleep()`和`wait()`时的运行结果如下  
![程序运行结果2](/blog_img/difference-between-interrupt-interrupted-isinterrupted/thread_interrupted_running_result_2.png "程序运行结果")  
分析运行结果可知：无论调用哪种方法，在中断线程时都会抛出`InterruptedException`异常，其运行结果都为false则意味着中断状态也不会设置。

以`JDK1.8`为例，关于调用`interrupt()`方法时中断状态的设置，可以从官网上看到如下描述：

>If this thread is blocked in an invocation of the wait(), wait(long), or wait(long, int) methods of the Object class, or of the join(), join(long), join(long, int), sleep(long), or sleep(long, int), methods of this class, then its interrupt status will be cleared and it will receive an InterruptedException.    
<br/>
>If this thread is blocked in an I/O operation upon an InterruptibleChannel then the channel will be closed, the thread's interrupt status will be set, and the thread will receive a ClosedByInterruptException.   
<br/>
>If this thread is blocked in a Selector then the thread's interrupt status will be set and it will return immediately from the selection operation, possibly with a non-zero value, just as if the selector's wakeup method were invoked.  
<br/>
>If none of the previous conditions hold then this thread's interrupt status will be set.  

综上，`interrupt()`、`intterupted()`、`isInterrupted()`这3个方法的区别与作用总结如下：

* **interrupt()**,中断调用该方法的当前线程，并设置其中断标记，具体而言分下述4种情况
	- 若当前线程被`wait()`、`join()`、`sleep()`这些方法(含超时)阻塞时，调用该方法中断线程时会清除中断标记，同时抛出`InterruptedException`异常；
	- 若当前线程被一个可中断的I/O操作阻塞，则会设置中断状态，同时关闭I/O通道并抛出`ClosedByInterruptException`异常；
	- 若当前线程被一个异步I/O操作(`Selector`)阻塞，则会设置中断状态并立即返回，看起来异步I/O操作的唤醒方法被调用；
	- 除以上3种情况外，其余情形下中断状态会被正常设置；
* **interrupted()**,检测当前线程是否被中断，如已被中断过则清除中断状态
* **isInterrupted()**,检测调用该方法的线程是否被中断，不清除中断标记

## 线程中断的使用
在Oracle官网有关于**[interrupt](https://docs.oracle.com/javase/tutorial/essential/concurrency/interrupt.html)**的如下如说明

>An interrupt is an indication to a thread that it should stop what it is doing and do something else. It's up to the programmer to decide exactly how a thread responds to an interrupt, but it is very common for the thread to terminate.

从上可知，当一个线程被中断时，JVM没有强制让当前线程立即中断,JVM只是要求我们在发生线程中断时应该停止当前正在执行的任务转而去执行其它任务。一个常用的场景是在循环体中利用`isInterrupt()`进行判断是否被中断，在没有被中断时在循环体中执行业务逻辑，当被中断时则跳出循环体然后执行其它业务逻辑，如下所示：
```java
while (!Thread.currentThread().isInterrupted()) {
	//do something when not been interrupted
}
//do other things when been interrupted
```

## 与LockSupport的比较
利用**[LockSupport](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/LockSupport.html)**方法重新修改上述代码，修改后的代码如下:

```java
public void testThread() {
	Thread threadA = new Thread(() -> {
		LockSupport.park();
		System.out.println(Thread.currentThread().isInterrupted());
		System.out.println(Thread.interrupted());
		System.out.println(Thread.currentThread().isInterrupted());
	}, "Thread_A");
	
	Thread threadB = new Thread(() -> {
		System.out.println(Thread.currentThread().getName() + " will unpark ThreadA");
		LockSupport.unpark(threadA);
	}, "Thread_B");
	
	threadA.start();
	try {
		Thread.sleep(30);
	} catch (InterruptedException e) {
		e.printStackTrace();
	}
	threadB.start();
}
```
程序运行结果如下：  
![程序运行结果3](/blog_img/difference-between-interrupt-interrupted-isinterrupted/thread_interrupted_running_result_3.png "程序运行结果")   
从上图中可以看出`LockSupport`中的方法来阻塞和唤醒线程时(更准确的说法是通过许可来觉得线程是否可运行)，既不修改线程的中断状态也不抛出`InterruptedException`异常。

参考文章：

* https://stackoverflow.com/questions/3590000/what-does-java-lang-thread-interrupt-do
* https://docs.oracle.com/javase/tutorial/essential/concurrency/interrupt.html
* https://www.ibm.com/developerworks/library/j-jtp05236/

关于如何处理`InterruptedException`异常，请参见本人翻译一篇文章**[[译]如何处理InterruptedException](/post/translate/dealing-with-interrupted-exception/)**