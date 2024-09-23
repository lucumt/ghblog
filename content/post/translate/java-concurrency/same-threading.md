---
title: "5.[译]同线程系统"
date: 2022-06-16T12:16:44+08:00
lastmod: 2022-06-16T12:16:44+08:00
draft: false
keywords: ["java concurrency"]
description: "本文介绍了将单个线程扩展的方式来实现并发执行"
tags: ["Java","Java Concurrency"]
categories: ["Java编程","翻译"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: false
postMetaInFooter: true
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to showS
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

本文翻译自[**Same-threading**](https://jenkov.com/tutorials/java-concurrency/same-threading.html)。

同线程系统(Same-threading)[^1] 是由一个单线程横向扩展为N个线程系统并行执行的并发模型。

同线程系统不是纯粹的单线程系统，因为它包含多个线程。 但其中每个线程都像单线程系统一样运，因此采用术语同线程而不是单线程。

<!--more-->

## 单线程和同线程设计视频教程

如果你更喜欢看视频，我也准备了一个相应的视频[**单线程和同线程设计**](https://www.youtube.com/watch?v=QrYIOs1dA3M&list=PLL8woMHwr36EDxjUoCzboZjedsnhLP1j4&index=22)。

{{< youtube "QrYIOs1dA3M" >}}

## 为什么选择单线程系统

你可能会很好奇为什么今天还有人设计并使用单线程系统，单线程系统之所以受欢迎，是因为它的并发模型比多线程系统简单的多，单线程系统并不同其它线程共享状态(数据/对象)，这使得单线程系统可以更好的利用非并发数据结构、CPU以及CPU缓存。

不幸的是单线程系统没有重复利用现代CPU的特性，一颗现代CPU通常带有 2, 4, 6, 8或更多的内核，每个内核在功能上都可以当做一个单独的CPU，如下图所示，单线程系统只能利用这些内核中的一个。

![单线程系统对于CPU利用不充分](/blog_img/java-concurrency/same-threading/same-threading-0.png "单线程系统对于CPU利用不充分")

## 同线程系统--单线程的横向扩展

为了利用CPU中的所有内核，可以通过横向扩展单线程系统来达到此目的。

### 每个CPU一个线程

同线程系统，通常在每个CPU中运行一个线程，如果一台计算机包含4个CPU或者一个CPU包含4个内核，那么运行4个相同的线程实例(4个单线程系统)是正常的，下图展示了这一原理：

![每个CPU一个线程](/blog_img/java-concurrency/same-threading/same-threading-0-1.png "每个CPU一个线程")  

## 无共享状态

由于一个同线程系统有多个线程在其中执行，其看起来与传统的单线程系统很相似，实际上它们有一些细微的差别。

同线程系统和传统多线程系统的主要区别是同线程系统中不同线程间不共享状态，没有并发访问的共享内存，也没有并发数据结构等可以用来实现线程间的数据共享，下图展示了这种区别

![同线程不共享数据](/blog_img/java-concurrency/same-threading/same-threading-4.png "同线程不共享数据")  

缺乏共享状态导致同线程系统的中每个线程看起来都像一个单线程系统，但是，由于一个同线程系统可以包含多个线程使得它们并不是真正的“单线程系统”。由于没有更好的名称，我认为将这样的系统称之为同线程系统(Same-threading )更准确，而不是“采用单线程设计的多线程系统”。

同线程系统在本质上意味着数据都在同一个线程中处理，并且没有线程并发的共享数据，这种场景有时称之为无共享状态并发或者隔离的状态并发。

## 负载分配

很显然，同线程系统需要在运行的单个线程之间分配工作负载，如果只有一个线程被分配工作负载，那么此系统实际上变成了一个单线程系统。

如何在不同的线程间精准的分配工作负载取决于你的系统设计，以下部分将介绍一些相关内容。

### 单线程微服务

如果系统由多个微服务组成，则每个微服务都能以单线程模式运行，当将多个微服务系统部署到一台服务器上时，其中的每个微服务都能以单线程的方式在单个CPU上执行。

微服务系统本质上不共享任何数据，因而是同线程系统的一个很好的实例。

### 服务间数据分片

如果系统间确实需要共享数据或者共享数据库，则可以对其进行分片，它意味着将数据在多个数据库之间进行划分。数据通常都被划分，以便彼此相关的数据都位于统一数据库中。例如，所有属于某个“所有者”的数据都将被插入同一个数据库中。不过，数据分片超出了本篇的范围，你需要自己去搜索相关教程。

## 线程间通信

若同线程系统中的线程间需要通信，可通过传递消息来实现。如果线程 A 想要向线程 B 发送消息，线程 A 可以通过生成消息（字节序列）来实现。 然后线程 B 可以复制该消息（字节序列）并读取它。 通过复制消息，线程 B 确保线程 A 在线程 B 读取消息时不能修改消息，复制后线程 A 无法访问消息副本。

下图展示了线程间通过消息传递实现通信

![线程间通过消息传递通信](/blog_img/java-concurrency/same-threading/same-threading-5.png "线程间通过消息传递通信")  

线程间通信可以通过队列、管道、unix 套接字、TCP 套接字等进行，只要适合您的系统。

## 更简单的并发模型

在同线程系统中每个线程中运行的系统都可以像单线程一样实现，这意味着其内部的线程共享模型比并发状态要更简单，因此也不用担心并发数据结构以及由此导致的各种问题。

## 图示说明

以下是单线程、多线程和同线程的示例图解，以便我们可以更容易的了解它们之间的区别。

首先展示的是单线程系统

![单线程系统](/blog_img/java-concurrency/same-threading/same-threading-1.png "单线程系统")  

第二幅图展示的是线程间共享数据的多线程系统

![线程间共享数据的多线程系统](/blog_img/java-concurrency/same-threading/same-threading-2.png "线程间共享数据的多线程系统")  

第三幅图展示了由2个包含隔离数据线程组成的同线程系统，它们之间通过传递消息进行通信

![同线程系统通过传递消息通信](/blog_img/java-concurrency/same-threading/same-threading-3.png "同线程系统通过传递消息通信")  

## 在Java中使用Thread Ops

Thread Ops for Java 是一个开源工具包，旨在帮助您更轻松地实现不同状态的同线程系统。 Thread Ops 包含用于启动和停止单个线程以及在单个线程中实现某种程度的并发的工具。 如果您对使用相同线程的应用程序设计感兴趣，那么看看 Thread Ops 可能会很有趣。 您可以在我的 Thread Ops for Java 教程中阅读有关 Thread Ops 的更多信息。



[^1]:注：原文为Same-threading，由于中文网络中没有合适的名词，故将其翻译为同线程系统，表示在一个系统中的相同相似的线程。
