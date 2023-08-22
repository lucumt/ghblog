---
title: "[译]JUnit 5 生命周期-注释之前和之后"
date: 2023-06-16T19:21:43+08:00
lastmod: 2023-06-16T19:21:43+08:00
draft: true
keywords: ["junit5"]
description: "翻译关于JUnit5生命周期的博文"
tags: ["junit5"]
categories: ["翻译","JUnit5翻译"]
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
centerImage: true
borderImage: false

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

本文翻译自[**JUnit 5 Test Lifecycle: Before and After Annotations**](https://www.arhohuttunen.com/junit-5-test-lifecycle/)。

<!--more-->

![JUnit 5 Chart Diagram](/blog_img/translate/junit5/junit-5-test-lifecycle/junit-5-chart-diagram.webp "JUnit 5 Chart Diagram") 

在本教程中我们将学习如何在测试类的每个测试方法或测试类中所有方法之前和之后运行代码，同时将了解嵌套测试和扩展测试时的执行顺序。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

# 生命周期相关方法

一个生命周期方法是指任何添加了`@BeforeAll`、`@AfterAll`、`@BeforeEach`或`@AfterEach`注解的方法，生命周期方法在实际测试方法之前或之后执行。

# 测试生命周期

![JUnit 5 Per Method Lifecycle](/blog_img/translate/junit5/junit-5-test-lifecycle/junit-5-per-method-lifecycle.svg "JUnit 5 Per Method Lifecycle") 

## 按类实例测试

![JUnit 5 Per Class Lifecycle](/blog_img/translate/junit5/junit-5-test-lifecycle/junit-5-per-class-lifecycle.svg "JUnit 5 Per Class Lifecycle") 

## 更具体的例子



# 嵌套测试生命周期

# 延长生命周期

# 总结

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-lifecyle)中找到。