---
title: "在Hugo生成的博客中动态的修改样式"
date: 2022-08-04T17:04:09+08:00
lastmod: 2022-08-04T17:04:09+08:00
draft: true
keywords: ["hugo"]
description: "基于hugo中的Even主题来动态的修改博客样式"
tags: ["hugo","Go"]
categories: ["个人博客"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
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
---

自己一直特别羡慕[**博客园**](https://www.cnblogs.com)上某些博主的博文样式（如[武培轩](https://www.cnblogs.com/wupeixuan/p/13450815.html),[JavaGuide](https://www.cnblogs.com/javaguide/p/16385150.html))，这些博文样式第一眼看起来就很清爽,让人很有阅读的欲望，而我总感觉自己博客的样式特丑陋。即使不关注内容，只看排版布局和样式，有时候自己写完一篇文章自己都不想去看，何况别人！



终于有一天我受不了自己的博客样式，决定对其升级一版，在这个过程中发现个人采用的博客主题[**Even**](https://github.com/olOwOlo/hugo-theme-even)功能很强大，基于上没做啥修改就达到了自己想要的效果，特此记录下。

<!--more-->

# 源码分析

个人博客基于[**Golang**](https://go.dev/)语言开发的[**Hugo**](https://gohugo.io/)最开始是采用[**hugo-redlounge**](https://github.com/tmaiaroto/hugo-redlounge/)主题来实现的，经过一段时间的使用之后感觉`hugo-redlounge`主题左侧会固定占用一部分宽度来展示个人信息，而这部分其实并无太大意义，故后来将其切换为如今的[**Even**](https://github.com/olOwOlo/hugo-theme-even)样式。

# 样式修改

## 修改方式



## 结果展示

### Default样式

### Mint Green样式

### Cobalt Blue样式

### Hot Pink样式

### Dark Violet样式