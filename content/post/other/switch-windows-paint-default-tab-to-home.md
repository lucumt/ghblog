---
title: "将Windows中的画图工具默认菜单从“文件”切换到“菜单”"
date: 2020-04-07T15:21:27+08:00
lastmod: 2020-04-07T15:21:27+08:00
draft: false
keywords: ["windows","mspaint","画布"]
description: "简要介绍如何将Windows中的画图工具默认菜单从“文件”切换到“菜单”"
tags: ["windows"]
categories: ["工具使用"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: 
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

简要介绍如何在`Windows`中的画图工具新打开一个画图时将其默认菜单从`文件`切换到`菜单`。

<!--more-->

## 问题

自己在工作中经常使用`Windows`系统自带的画图工具来编写图示说明文档，个人主要是通过`Win`+`R`打开`运行`窗口，之后输入`mspaint`，最开始其打开的界面类似如下，默认打开的是`主页`，可直接使用其上的各种工具，非常方便。

![画图工具正常打开时的界面](/blog_img/other/switch-windows-paint-default-tab-to-home/mspaint-home-tab.png "画图工具正常打开时的界面") 

突然某一天不知道发生了什么事情，利用`mspaint`打开画图后呈现出的界面类似如下，默认打开的是`文件`菜单，每次要对图片进行操作时都需要手工切换到`主页`菜单，操作完毕后它又会自动切换回`文件`菜单，使用起来很不方便。

![画图工具以文件菜单方式打开](/blog_img/other/switch-windows-paint-default-tab-to-home/mspaint-file-tab.png "画图工具以文件菜单方式打开") 

## 解决方法

一开始自己在百度上搜索后没有找到自己想要的答案，只能每次手工切换到`主页`菜单，但随着使用次数的增多，我有些受不了了，**难道我真的是一个这么将就的人？** 我决心解决这个问题！



百度不行，咱就上Google，一番搜索后终于在[这篇文章](https://answers.microsoft.com/en-us/windows/forum/all/paint-default/284ac7fb-a8d5-4786-afe8-81da0950cef3)中找到答案

> Paint does not have a File Tab, it only has a File Menu, that is always highlighted Blue in Paint
> What happens sometimes is the Home tab becomes hidden, double click the Home Tab to make that open permanently
> Then restart Paint to see if it then defaults to an open Home Tab . . .

在下面的回复中有这么一句话

> Worked like a charm! And so simple.... Thank you Dave.

看到它我就知道这个答案很靠谱了，试了下果然生效！

操作方法也很简单，如下图所示，双击`主页`菜单然后重新打开画图工具即可！

需要注意的是若默认情况下已经打开`主页`菜单，再次双击该菜单时，会将`主页`菜单隐藏，自己之前的问题或许就是这个导致的。

![双击“主页”切换回原有设置](/blog_img/other/switch-windows-paint-default-tab-to-home/mspaint-home-tab-double-click.png "双击“主页”切换回原有设置") 

