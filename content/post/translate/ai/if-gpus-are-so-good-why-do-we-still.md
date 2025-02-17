---
title: "[译]既然GPU如此强大，为啥我们还需要使用CPU?"
date: 2025-02-17T11:21:37+08:00
lastmod: 2025-02-17T11:21:37+08:00
draft: true
keywords: ["CPU","GPU"]
description: ""
tags: ["AI"]
categories: ["翻译","人工智能"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: true
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
codeTabSeperator: "::"
moreMeta: true

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

本文翻译自[**If GPUs Are So Good, Why Do We Still Use CPUs At All?**](https://codingstuff.substack.com/p/if-gpus-are-so-good-why-do-we-still)

<!--more-->

最近Twitter上出现了一段2009年的老视频，它旨在让观众更直观地了解CPU和GPU之间的区别。

可观看下述时长90秒的视频以了解更多:

{{< youtube -P28LKWTzrI >}}

其主要让CPU和GPU进行一场面对面的绘画比赛，相关处理器连接到发射彩蛋的机器上。

CPU需要整整30秒才能绘制一个非常基本的笑脸：

![CPU绘画结果](/blog_img/translate/ai/if-gpus-are-so-good-why-do-we-still/cpu-paint-result.webp "CPU绘画结果") 

而GPU却在瞬间画出了蒙娜丽莎：

![GPU绘画结果](/blog_img/translate/ai/if-gpus-are-so-good-why-do-we-still/gpu-paint-result.webp "GPU绘画结果") 