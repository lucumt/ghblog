---
title: "volatile关键字在Java程序中的使用"
date: 2018-12-08T23:49:33+08:00
lastmod: 2018-12-08T23:49:33+08:00
draft: true
keywords: ["volatile","Java Concurrency"]
description: "Volatile关键字在Java程序中的使用"
tags: ["Java"]
categories: ["Java编程","Java多线程"]
author: "Rosen Lu"

---

`volatile`关键字在Java多线程编程中很常见，由于自己之前学习多线程时一度以为只要需确保线程可见性的代码都需要使用`volatile`关键字，后来发现并不是这样的，故简单记录下。

<!--more-->
### 典型使用场景

下面这段代码创建了两个线程threadA和threadB，threadA中运行display()方法，threadB中运行stop()方法，在threadA启动1秒后启动threadB。

在stop布尔变量上分别注释和不注释`volatile`的运行结果如下：

* 注释`volatile`关键字时，display()方法一直运行，程序不能终止；
* 不注释`volatile`关键字时，display()方法会在调用stop()方法后立即停止运行，程序终止；

注: 根据实际运行电脑的配置，有可能不注释`volatile`关键字时调用stop()方法也能让display()方法立即停止运行。 

代码清单1: `volatile`关键字的使用
```java
public class VolatileTest {

	//注释掉volatile关键字时display()方法会一直运行下去
	private /*volatile*/ boolean stop = false;
	
	private void stop() {
		stop = true;
	}
	
	private void display() {
		while(!stop) {
		}
		System.out.println(LocalDateTime.now());
	}
	
	public void test() {
		Thread threadA = new Thread(()->{
			this.display();
		});
		Thread threadB = new Thread(()->{
			this.stop();
		});
		
		threadA.start();
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		threadB.start();
	}
	
	public static void main(String[] args) {
		VolatileTest vt = new VolatileTest();
		vt.test();
	}
}
```

上述代码展示了`volatile`的典型使用场景:  

**当有两个或以上的线程同时读取一个共享的可变变量时，为了确保每个线程都能获取到最新的值，应该使用`volatile`关键字对该变量进行修饰，确保该变量的可见性。**

关于`volatile`如何实现确保变量可见性，网上已经有很多资料，请自行查阅，如[volatile和lock原理分析](https://liuzhengyang.github.io/2017/03/28/volatileandlock/)。

### 有独占锁时不需要使用volatile

既然`volatile`变量确保的是在多个线程**同时读取**一个变量时确保内存可见性，如果有多个线程对共享变量进行读写时，若由于**排它锁(exclusive lock)**的存在导致任一时刻只能有一个线程对共享变量进行读写操作，此时是否还需要添加`volatile`关键字呢？答案是否定的，从字面意思可以看出，由于同一时刻只能有一个线程访问变量，所以变量可见性的问题不会存在，故没必要添加`volatile`关键字。

以Java8为例，在Java语言规范中有关于`volatile`的说明 **[volatile Fields](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.3.1.4)**中有如下片段:

>The Java programming language allows threads to access shared variables (§17.1). As a rule, to ensure that shared variables are consistently and reliably updated, a thread should ensure that it has exclusive use of such variables by obtaining a lock that, conventionally, enforces mutual exclusion for those shared variables.  
<br/>
>The Java programming language provides a second mechanism, volatile fields, that is more convenient than locking for some purposes.  
<br/>
>A field may be declared volatile, in which case the Java Memory Model ensures that all threads see a consistent value for the variable (§17.4).

从这段文字可以看出`volatile`相对于独占锁提供了更加简便的方式让变量一致且可靠的更新，`volatile`只确保读取的值正确，对写入影响不大，而独占锁则限制了同时只能有一个读或写，导致出现性能问题以及在特定的场景下无法使用的问题(如代码清单1所示，若采用独占锁则同时只能有一个线程运行，违背了设计初衷)。

下述代码展示了一个典型的生产者/消费者模式，由于独占锁的存在，即使没有使用`volatile`关键字，程序也能正常工作。  
  
代码清单2: 生产者/消费者模式
```java
public class ProducerConsumerTest {

	private static final int MAX_SIZE = 10;

	//这两个变量没必要使用volatile关键字修饰
	private LinkedList<String> items = new LinkedList<>();
	private int count;

	private Object lock = new Object();

	private void produce(String ele) {
		synchronized (lock) {
			try {
				while (count >= MAX_SIZE) {
					lock.wait();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			items.add(ele);
			count++;
			System.out.println(Thread.currentThread().getName() + " 添加了元素  " + ele + "  当前元素总数为 " + count);
			lock.notifyAll();
		}
	}

	private String consume() {
		String result = null;
		synchronized (lock) {
			try {
				while (count == 0) {
					lock.wait();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			result = items.pollLast();
			count--;
			System.out.println(Thread.currentThread().getName() + " 获取了元素  " + result + "  当前元素总数为 " + count);
			lock.notifyAll();
		}
		return result;
	}
	
	public void test() {
		for(int i=0;i<10;i++) {
			new Thread(() -> {
				for(int j=0;j<5;j++) {
					String ele = UUID.randomUUID().toString().replace("-", "");
					this.produce(ele);
				}
			},"Producer_" + i).start();
		}
		for(int i=0;i<5;i++) {
			new Thread(() -> {
				for(int j=0;j<10;j++) {
					this.consume();
				}
			},"Consumer_" + i).start();
		}
	}
	
	public static void main(String[] args) {
		new ProducerConsumerTest().test();
	}
}
```

### 利用volatile在单例模式中实现双重检查

`volatile`关键字不仅可以确保线程可见性，还能禁止重排序，它的一个典型应用是利用双重检查实现线程安全的单例设计模式，如代码清单3所示。在该程序中主要利用了`volatile`禁止重排序的功能，详细说明请参见[Java并发编程的艺术](https://item.jd.com/11740734.html)P67中的 *双重检查锁定与延迟初始化* 。

代码清单3: 利用双重检查实现线程安全的单例模式
```
public class Singleton {

	private static volatile Singleton instance = null;
	
	private Singleton() {
	}
	
    public static Singleton getInstance() {
    	if(instance == null) {
    		synchronized(Singleton.class) {
    			if(instance == null) {
    				instance = new Singleton();
    			}
    		}
    	}
    	return instance;
    }
}
```

参考文章:

* [Java并发编程的艺术](https://item.jd.com/11740734.html)
* [volatile和lock原理分析](https://liuzhengyang.github.io/2017/03/28/volatileandlock/)
* [Simplest and understandable example of volatile keyword in java](https://stackoverflow.com/questions/17748078/simplest-and-understandable-example-of-volatile-keyword-in-java)
* [Difference between volatile and synchronized in Java](https://stackoverflow.com/questions/3519664/difference-between-volatile-and-synchronized-in-java)