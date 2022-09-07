---
title: "Java中不提供ConcurrentArrayList的原因分析"
date: 2021-02-07T22:05:43+08:00
lastmod: 2021-02-07T22:05:43+08:00
draft: true
keywords: []
description: ""
tags: ["Java","Java Concurrency"]
categories: []
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
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

<!--more-->

Java并发编程中的 [**ConcurrentHashMap**](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html)是一个常用的数据结构，也是面试中的一个高频考点，它是`HashMap`在并发模式下的实现，那为啥`ArrayList`没有对应的ConcurrentArrayList呢，本文试着分析下。