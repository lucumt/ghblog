---
title: "加快Spring Boot的启动速度"
date: 2019-06-08T17:19:54+08:00
lastmod: 2019-06-08T17:19:54+08:00
draft: true
keywords: ["spring boot","配置","启动速度"]
description: "通过去掉各种无用的启动配置来加快Spring Boot的启动速度"
tags: ["spring","spring-boot"]
categories: ["spring系列","spring-boot"]
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

之前面试时被问到过此题，基于网上资料以及个人理解，简单记录下如果根据实际项目需求通过去掉各种无用的配置来缩短`Spring Boot`的启动时间。

<!--more-->

参考文档:

1. https://www.helloworld.net/p/2393829294
2. https://segmentfault.com/a/1190000042204196