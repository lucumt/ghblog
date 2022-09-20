---
title: "将3个小int整数合并到1个long中"
date: 2022-09-08T10:04:59+08:00
lastmod: 2022-09-08T10:04:59+08:00
draft: true
keywords: ["bit","int","long"]
description: "将3个小int整数合并到1个long中"
tags: ["bit"]
categories: ["位操作","Java编程"]
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
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true

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

近期在工作中遇到由于`HTTP`返回的内容较多导致系统响应延迟的问题，最终自己结合[gzip](https://en.wikipedia.org/wiki/Gzip)、[Protocol Buffers](https://developers.google.com/protocol-buffers)、[位运算](https://zh.wikipedia.org/wiki/%E4%BD%8D%E6%93%8D%E4%BD%9C)等将`HTTP`响应返回的数据从15M减少到1M从而解决系统无卡顿问题。其中对于`位运算`部分自己是结合业务实际，将3个小型`int`转化为1个`long`，将数据量减少三分之一，简单记录下其实现(以`Java`实现为例)。

<!--more-->

# 背景

某个项目模块要将一系列的坐标数据返回给前端页面，坐标数据由x、y、z这3个维度来定义且用`float`表示，它们的约束条件如下：

> $|x|$<150、$|y|$<50、$|z|$<50

示例文件如下：

```bash
123.45	34.56	23.78
111.35	-32.56	21.78
103.77	24.83	-13.78
#...共有超过10万行数据
99.83	21.35	43.39
```

最开始自己是采用自己是将它们逐行读取并以`String`的形式封装到`JSON`中返回给前端，此种原始的方式很明显会导致返回给前端的数据量巨大从而严重影响性能。

一开始自己从如下几个方面着手优化：

1. 将返回的数据从`String`转化为`float`

接下来自己集合`Protocol Buffers`与`gzip`

# 分析

![int和long占用的bit位数](/blog_img/bit/merge-three-int-number-into-a-long-number/bits-used-by-int-and-long.png "int和long占用的bit位数")  

# 实现

完整代码参见[**SmallIntBitCompressTest.java**](https://github.com/lucumt/myrepository/blob/master/bit/SmallIntBitCompressTest.java)