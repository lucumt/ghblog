---
title: "在KubeSphere的流水线中下载构建具有子模块的项目"
date: 2023-09-07T14:27:13+08:00
lastmod: 2023-09-07T14:27:13+08:00
draft: true
keywords: ["devops","jenkins","git","kubesphere","submodule","子模块"]
description: "记录如何在KubeSphere的流水线中下载构建具有子模块的项目"
tags: ["devops","jenkins","git","kubesphere"]
categories: ["工具使用","持续集成"]
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

在使用[**KubeSphere**](https://www.kubesphere.io/zh/)对公司项目进行持续集成时，有一个项目在`Gitlab`中采用了[**Submodules**](https://git-scm.com/book/en/v2/Git-Tools-Submodules)结构，导致原有的流水线不能直接使用，一番折腾后找到两种方案，其中一种比较优雅，简单记录下。

<!--more-->

# 没有子模块

在没有子模块时，之后自动生成的`Jenkins`代码类似如下

```groovy
stage('拉取代码') {
    agent none
    steps {
        git(credentialsId: 'gitlab-account', url: 'http://gitlab.xxx.com/lucumt-group/system.git', branch: '$BRANCH_NAME', changelog: true, poll: false)
    }
}
```



# 有子模块时

## 多次下载

## 递归下载

