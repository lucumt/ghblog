---
title: "多个docker-compose.yml文件容器冲突的原因分析"
date: 2023-07-25T22:53:33+08:00
lastmod: 2023-07-25T22:53:33+08:00
draft: true
keywords: []
description: ""
tags: ["docker"]
categories: ["容器化"]
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

<!--more-->

参考文章 https://stackoverflow.com/questions/43126022/docker-is-not-creating-new-container-but-recreates-running-one

环境变量相关说明https://docs.docker.com/compose/environment-variables/envvars/

解决方案：

1. 显示指定container_name
2. 设置不同的service name
3. 放到不同的文件夹下，确保路径不相同