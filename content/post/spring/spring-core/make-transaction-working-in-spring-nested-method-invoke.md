---
title: "Spring中方法嵌套调用导致的事务不生效的解决方案"
date: 2019-09-25T17:05:32+08:00
lastmod: 2019-09-25T17:05:32+08:00
draft: true
keywords: ["spring","事务","方法嵌套调用."]
description: "Spring中方法嵌套调用导致的事务不生效的解决方案"
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

结合`Spring`相关源码来说明在`Spring`中存在方法嵌套调用时被嵌套的方法中事务不生效的原因以及相关的解决方案。

<!--more-->
