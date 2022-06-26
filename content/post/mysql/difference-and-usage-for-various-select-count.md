---
title: "MySQL中不同SELECT COUNT统计总数时的区别"
date: 2022-06-25T17:18:27+08:00
lastmod: 2022-06-25T17:18:27+08:00
draft: true
keywords: ["mysql","select count"]
description: "记录mysql中select count(id)、select count(*)以及select count(1)的区别"
tags: ["mysql"]
categories: ["数据库"]
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

在数据库中使用`COUNT`函数统计总数是常用操作，本文参考网上资料以及个人实际操作记录下`MySQL`中通过`COUNT(id)`、`COUNT(*)`以及`COUNT(1)`的区别以及使用场景。

<!--more-->
