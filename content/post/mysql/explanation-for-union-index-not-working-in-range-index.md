---
title: "联合索引中范围查询导致索引部分生效"
date: 2019-05-22T13:28:26+08:00
lastmod: 2019-05-22T13:28:26+08:00
draft: true
keywords: ["mysql","联合索引","范围查询"]
description: "基于个人理解说明联合索引中范围查询导致索引部分生效的原因"
tags: ["mysql"]
categories: ["数据库"]
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

`MySQL`中索引相关的知识是数据库面试高频题，本文以图文结合的方式简要说明在联合索引中使用范围查询时索引只有部分生效的原因。

<!--more-->
