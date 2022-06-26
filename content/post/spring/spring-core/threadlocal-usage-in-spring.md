---
title: "ThreadLocal在Spring中的使用场景分析"
date: 2022-06-25T17:07:18+08:00
lastmod: 2022-06-25T17:07:18+08:00
draft: true
keywords: ["spring","threadlocal"]
description: "简要ThreadLocal在Spring中的使用场景分析"
tags: ["spring"]
categories: ["spring系列"]
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

`ThreadLocal`在多线程环境中用于确保变量实现线程间隔离，本文通过源码分析的方式来展示`ThreadLocal`在`Spring`中的典型使用场景。

<!--more-->
