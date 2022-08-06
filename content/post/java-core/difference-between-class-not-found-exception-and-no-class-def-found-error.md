---
title: "ClassNotFoundException与NoClassDefFoundError对比"
date: 2019-01-02T14:18:46+08:00
lastmod: 2019-01-02T14:18:46+08:00
draft: true
keywords: ["ClassNotFoundException","NoClassDefFoundErro"]
description: "简要说明ClassNotFoundException与NoClassDefFoundError对比"
tags: ["Java"]
categories: ["Java编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
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



[**ClassNotFoundException**](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassNotFoundException.html)与[**NoClassDefFoundError**](https://docs.oracle.com/javase/8/docs/api/java/lang/NoClassDefFoundError.html)是`Java`开发中经常会遇到的异常与错误，本文基于个人工作中遇到的场景以及网上的资料，简要总结它们的差异、出现场景以及规避方案。

<!--more-->

# 分析

## ClassNotFoundException

![ClassNotFoundException类结构](/blog_img/java-core/difference-between-class-not-found-exception-and-no-class-def-found-error/class-not-found-exception-structure.png "ClassNotFoundException类结构") 

上图为`ClassNotFoundException`的类继承结构，从图中可看出它继承自[Exception](https://docs.oracle.com/javase/8/docs/api/java/lang/Exception.html)且没有继承自[RuntimeException](https://docs.oracle.com/javase/8/docs/api/java/lang/RuntimeException.html),基于`Java`语言规范它属于`Checked Exception`[^1]，实际编程时必须在代码中利用`try-catch`显示的捕获处理或者利用`throws`将其抛出到上一层调用者处理。

在`ClassNotFoundException`的[官方API](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassNotFoundException.html)中对于其产生的原因有如下描述:

> Thrown when an application tries to load in a class through its string name using:
>
> - The `forName` method in class `Class`.
> - The `findSystemClass` method in class `ClassLoader` .
> - The `loadClass` method in class `ClassLoader`.
>
> but no definition for the class with the specified name could be found.

基于上述描述可知`ClassNotFoundException`产生的根本原因如下：

> **在应用程序类中通过对应类的字符串名称去加载该类时找不到类的定义，即会抛出此异常**

在实际场景中可分为两类:

1. 在`Class`类中通过`forName`方法去加载时找不到类
2. 在`ClassLoader`类中通过

## NoClassDefFoundError



# 总结

|                | **ClassNotFoundException** | NoClassDefFoundError |
| :------------- | :------------------------: | :------------------: |
| 继承关系       |                            |                      |
| 类型           |            异常            |         错误         |
| 程序是否可恢复 |                            |                      |



参考:

* https://segmentfault.com/a/1190000021292121

* https://stackoverflow.com/questions/28322833/classnotfoundexception-vs-noclassdeffounderror

[^1]: https://docs.oracle.com/javase/specs/jls/se7/html/jls-11.html