---
title: "在Hugo中利用Prism进行代码高亮"
date: 2024-07-30T18:31:44+08:00
lastmod: 2024-07-30T18:31:44+08:00
draft: true
keywords: ["hugo","prism","chroma","highlight","代码高亮"]
description: "简要介绍如何使用Prism来替换Hugo博客默认的代码高亮实现Chroma，实现支持更多的语言和更好的识别"
tags: ["hugo"]
categories: ["个人博客"]
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

简要介绍如何在基于[**Even**](https://github.com/olOwOlo/hugo-theme-even)主题的[**Hugo**](https://gohugo.io)博客中采用[**Prism**](https://prismjs.com)替换默认的[**Chroma**](https://github.com/alecthomas/chroma)进行代码高亮显示，实现支持更多的语言和更好的识别。

<!--more-->

## 背景

个人博客采用`Hugo`编写已经有年头了，由于`Hugo`是基于`Golang`编写的，而`Chroma`也是基于`Golang`编写的，故`Hugo`采用了`Chroma`作为其默认的代码高亮显示实现[^1]。自己的个人博客搭建完毕之后一直采用该默认实现。

之后的几年虽然自己的博客主题切换成了`Even`主题，也伴随着各种优化，但对于代码高亮这块却一直没变化，一方面是自己觉得其显示效果还行，没找到更适合自己口味的显示样式，就和代码开发一样，能正常运行的代码就不要随便优化重构(虽然自己很喜欢代码重构)，另一方面则是由于代码高亮切换，改动时涉及的底层页面较多。

近期由于部门内部要搭建基于`GitBook`的知识库，在此过程中采用的语法高亮为`Prism`，经过一段时间的时候之后个人感觉`Prism`对于代码高亮显示的支持更强，对它们的关键指标进行对比后，初步决定采用`Prism`替换`Chroma`。

|                |              Prime               |                Chroma                |
| -------------- | :------------------------------: | :----------------------------------: |
| **GitHub地址** | https://github.com/PrismJS/prism | https://github.com/alecthomas/chroma |
| **Star数**     |              12.2k               |                 4.3k                 |
| **支持语言数** |               297                |                 249                  |
| **插件支持**   |                是                |                  否                  |
| **渲染性能**   |                中                |                  高                  |
| **模块化**     |                是                |                  否                  |

在此过程中自己也参考了其它的一些高亮方案，尤其是[这篇文章](https://frostming.com/2020/12-08/highlight-lexer-comparison)通过表格形式详细的对比常用的代码高亮框架，虽然有比`Prism`更强的实现，但`Prism`已经基本上满足自己的要求，同时自己在`GitBook`的搭建过程中也积累了一定的使用经验，故最终决定采用`Prism`替换`Chroma`。

## 操作步骤

### 添加行号

### 代码复制

## 效果对比

`Java`代码显示结果对比如下

![Java显示效果对比](/blog_img/hugo/using-prismjs-to-highlight-code/hugo_java_code_compare.png "Java显示效果对比")

`JavaScript`代码显示结果对比如下

![JavaScript显示效果对比](/blog_img/hugo/using-prismjs-to-highlight-code/hugo_js_code_compare.png "JavaScript显示效果对比")

可看出整体上`Prismjs`的显示效果与识别度整体上比`Chroma`要更好。

[^1]: https://gohugo.io/content-management/syntax-highlighting