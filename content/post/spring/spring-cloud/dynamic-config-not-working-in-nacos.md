---
title: "Spring Cloud采用Nacos时外部配置文件不生效的原因分析"
date: 2022-06-25T17:27:44+08:00
lastmod: 2022-06-25T17:27:44+08:00
draft: true
keywords: ["spring","nacos","config file"]
description: "分析自己项目中采用Nacos遇到的动态配置不生效的问题以及解决方案"
tags: ["spring","spring-cloud","nacos"]
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

最近公司的新项目中采用了基于`Spring Cloud`的微服务架构，其中在基于`Nacos`进行配置时遇到了配置文件不生效的问题，基于网上自己以及自己对`Spring Boot`和`Nacos`源码的分析，最后找到了原因，故记录下。

主要是要放到到`boostrap.yml`文件而不是`application.yml`文件中，只有这样才能生效。

<!--more-->
