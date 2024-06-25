---
title: "Dockerfile中RUN、CMD与ENTRYPOINT的使用比较"
date: 2021-02-20T15:00:08+08:00
lastmod: 2021-02-20T15:00:08+08:00
draft: true
keywords: ["docker","dockerfile","RUN","CMD","ENTRYPOINT"]
description: "通过示例子简要说明Dockerfile中RUN、CMD与ENTRYPOINT的使用比较"
tags: ["docker"]
categories: ["容器化","翻译"]
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

本文翻译自[**Docker RUN vs CMD vs ENTRYPOINT**](https://codewithyury.com/docker-run-vs-cmd-vs-entrypoint/)。

<!--more-->

`Dockerfile`使用时的`CMD`与`ENTRYPOINT`指令其作用很相似，经常让人容易混淆，网络上有不少关于它们对比说明的文章，但个人感觉都不直观明晰。

本来自己想专门写一篇关于它们差异的博文，后来在网上发现国外有个作者已经写了类似的文章，同时读了之后有醍醐灌顶之感，故出于对作者的尊重与感谢，决定直接将其翻译成中文！

# 概要说明

# Docker镜像与分层

# Shell和Exec模式

## Shell模式

## Exec模式

## 如何运行bash脚本

# RUN指令

# CMD指令

# ENTRYPOINT指令

## Exec模式

## Shell模式

# 总结

