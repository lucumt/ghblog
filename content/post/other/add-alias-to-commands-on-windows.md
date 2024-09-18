---
title: "给Windows中的命令添加别名"
date: 2024-06-12T13:26:28+08:00
lastmod: 2024-06-12T13:26:28+08:00
draft: false
keywords: ["Windows","命令行","别名"]
description: "简要说明如何给Windows中的命令添加别名以方便使用"
tags: ["windows"]
categories: ["工具使用"]
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

简要记录如何在`Windows`操作系统中通过[**批处理脚本**](https://en.wikipedia.org/wiki/Batch_file)来给复杂的命令添加别名以简化使用。

<!--more-->

## 背景

自己使用`Hugo`进行博客开发已经有年头了，最近由于需要搭建新的网站[^1]需要使用特定的模板，而该模板对`Hugo`的版本有要求，故对`Hugo`版本进行了升级，从`v0.100`升级到`v0.126`。

升级之后虽然在搭建新网站时没有问题，但个人发现新版本的`Hugo`有一个特性让人用起来不太方便：

通过`hugo server -w -D`在本地启动`Hugo`环境时，默认会在本地`public`文件夹下生成全部的静态文件，从而导致通过`Git`提交时会提交大量无用的文件，在国内这种网络环境下容易`push`失败。

而之前自己使用`Hugo`时，若要手工生成静态文件需要在参数中显示指定，类似`hugo server --renderToDisk`，升级后的版本将其作为了默认选项。

进一步查看资料，发现该特性是在`v0.123`版本中引入的[^2]，具体原因参见[^3]，在该版本之后若想默认不在`public`目录下生成静态文件则需要在启动时手工添加相关指令，类似`hugo server -w -D --renderToMemory`。

尽管`Hugo`作者这么做肯定是有其原因的，但我习惯了只有在托管到`GitHub Pages`时才生成静态文件，同时我也是一个懒人，每次使用时都要添加`--renderToMemory`参数对我来说简直是一种折磨。

能否仿照`Linux`中的[**alias**](https://www.runoob.com/linux/linux-comm-alias.html)指令对前述命令进行封装，从而简化使用呢？

## 分析

我们知道在`Windows`的默认附带了很多指令，如`mspaint`、`mstsc`、`calc`等指令，可在运行窗口或`cmd`中输入这些指令直接使用，其原因为这些指令都位于`C:\Windows\System32`目录下，而该目录是系统目录，在启动时会进行加载。

而对于非`Windows`系统自带指令或软件，在我们安装过程中通常会提示我们是否要将该软件添加到`Path`环境变量中去，典型的如`Python`，当选择添加到`Path`环境变量中后，可重新打开一个`cmd`窗口，然后输入`python`指令进行相关的操作。即我们如果想要在`cmd`中能直接使用某个自定义指令，则该指令必须添加到`Path`环境变量中去。

在`Linux`中有`bash`、`zsh`等脚本可用于封装复杂的指令，同样在`Windows`中可使用`batch`脚本来封装复杂的指令，最终问题变为如下：

是否可通过`batch`脚本封装复杂的`Hugo`指令，并将该脚本添加到`Path`环境变量中来实现在`cmd`中直接使用封装后的指令？

## 实现

理论分析完毕后，接下来进行实际操作验证：

1. 新建一个名为`hugo_server.bat`的脚本，内容如下

   ```powershell
   @echo off
   hugo server -w -D --renderToMemory
   ```

2. 仿照下图，将`hugo_server.bat`的位置添加到`Path`环境变量中去

   ![将脚本路径添加到环境变量中去](/blog_img/other/add-alias-to-commands-on-windows/add-batch-script-to-path-enviroment.png "将脚本路径添加到环境变量中去") 

3. 重新打开一个`cmd`窗口，输入`hugo_server`(`batch`脚本的名称)，显示结果如下，可看出`hugo_server`别名正常工作，目的达到！

   ![bat脚本别名正常工作](/blog_img/other/add-alias-to-commands-on-windows/batch-script-works.png "bat脚本别名正常工作") 

[^1]:参见[创建多个GitHub Pages并且利用GoDaddy分别配置子域名](/post/github/create-multiple-github-pages-and-config-godaddy-subdomain/)
[^2]: 参见[Some breaking changes planned for Hugo 0.123.0](https://github.com/gohugoio/hugo/issues/11455)
[^3]:[Make the server flag --renderToDisk into --renderToMemory](https://github.com/gohugoio/hugo/issues/11987)