---
title: "Spring中@Autowire、@Ressource和@Component注解的比较"
date: 2022-08-21T13:15:07+08:00
lastmod: 2022-08-21T13:15:07+08:00
draft: true
keywords: ["spring","annotation","autowire","resource","component"]
description: "Spring中@Autowire、@Ressource和@Component注解的比较"
tags: ["spring","spring-boot"]
categories: ["spring系列"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
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

总结下`Spring`中最常见的3种依赖注入标签`@Component`、`@Autowire`、`@Resource`之间的区别。

<!--more-->

参考文档:

1. https://www.cnblogs.com/vipstone/p/16634716.html