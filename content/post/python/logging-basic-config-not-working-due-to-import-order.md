---
title: "Python中logging模块由于错误的import顺序导致basicConfig失效"
date: 2023-04-22T16:53:05+08:00
lastmod: 2023-04-22T16:53:05+08:00
draft: true
keywords: ["python","logging"]
description: ""
tags: ["python"]
categories: ["python编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: false
borderImage: true

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

简要记录下`Python`中`logging`模块由于在其它模块中错误的import顺序导致`basicConfig`失效的问题分析与解决。

**2023/09/08此问题未能复现，暂不继续**

<!--more-->

## 问题

在[**Python程序中使用Nacos**](/post/nacos/integrate-python-program-to-nacos/)这篇文章中整合`Nacos`时由于需要输出日志，中间需要打印日志，`Python`中的日志主要基于`logging`模块，默认情况下 `logging`模块生成的log打印出的日志格式不符合自己需求，于是自己想通过`basicConfig`设置实现自定义格式。

程序结构如下：

相关代码如下：

## 分析 & 解决

