---
title: "Java中WAITING和TIMED_WAITING状态的区别"
date: 2020-08-05T09:28:43+08:00
lastmod: 2020-08-05T09:28:43+08:00
draft: true
keywords: ["Java多线程"]
description: "Java中WAITING和TIMED_WAITING状态的区别"
tags: ["Java","Java Concurrency"]
categories: ["Java编程","Java多线程"]
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

简要对比`Java`中[WAITING](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.State.html#WAITING)和[TIMED_WAITING](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.State.html#TIMED_WAITING)的区别与使用场景。

<!--more-->



参考：

1. https://www.oreilly.com/library/view/java-threads-second/1565924185/ch04s04.html