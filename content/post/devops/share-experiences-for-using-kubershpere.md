---
title: "Kubershpere使用心得"
date: 2022-08-06T18:00:08+08:00
lastmod: 2022-08-06T18:00:08+08:00
draft: true
keywords: ["kubesphere","Jenkins","持续集成","持续部署"]
description: ""
tags: ["kubesphere"]
categories: ["持续集成"]
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
---

目前公司的开发方式都是手工编译&部署，十分低效，最近将Web开发相关的项目都基于[**KubeSphere**](https://kubesphere.io/)通过流水线方式实现了自动部署，在此过程中遇到了一些阻塞点，简单记录下它们的解决方案。

<!--more-->

# 流水线设计

基于部门现状以及参考网上资料，将产品部署环境划分为如下4种类型，各自的端口和作用如下：

|   环境   | 端口 | 作用           |
| :------: | :--: | -------------- |
| **dev**  |      | 本地开发阶段   |
| **sit**  |      | 各模块联调阶段 |
| **test** |      | 测试阶段       |
| **prod** |      | 产品部署阶段   |

虽然通过上述划分方式可有效的避免不同环境的互相干扰，但由于目前部门大部分产品都采用了[微服务](https://zh.wikipedia.org/zh-cn/%E5%BE%AE%E6%9C%8D%E5%8B%99)实现，每个微服务模块都有单独的代码管理，导致

![项目架构图](/blog_img/devops/share-experiences-for-using-kubershpere/project-architecture.png "项目架构图")  

# 脚本说明

## shell脚本

## scripti脚本

# 代码编译

## 前端

## 后端

# K8S容器探活

## 前端

## 后端

# 流水线实现

