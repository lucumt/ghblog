---
title: "Maven编译时遇到java.lang.ClassNotFoundException: org.apache.maven.shared.filtering.MavenFilteringException的问题解决"
date: 2023-07-26T15:07:08+08:00
lastmod: 2023-07-26T15:07:08+08:00
draft: true
keywords: []
description: ""
tags: ["java","maven"]
categories: ["java编程"]
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

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

简单记录在利用`mvn clean compile`时遇到的`java.lang.ClassNotFoundException: org.apache.maven.shared.filtering.MavenFilteringException`导致编译失败的问题解决。

<!--more-->

报错截图1：

![mvn compile执行报错](/blog_img/maven/class-not-found-exception-when-compile/mvn-compile-error-1.png "mvn compile执行报错")

报错截图2：

![mvn compile执行报错](/blog_img/maven/class-not-found-exception-when-compile/mvn-compile-error-2.png "mvn compile执行报错")