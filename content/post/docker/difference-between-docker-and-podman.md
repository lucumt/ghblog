---
title: "[译]Docker和Podman的差异"
date: 2024-04-26T10:46:34+08:00
lastmod: 2024-04-26T10:46:34+08:00
draft: true
keywords: []
description: ""
tags: ["docker"]
categories: ["容器化","翻译"]
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
centerImage: true
borderImage: false

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

本文翻译自[**Podman vs Docker: What are the differences?**](https://www.imaginarycloud.com/blog/podman-vs-docker/)

<!--more-->

![Docker和Podman的差异](/blog_img/docker/difference-between-docker-and-podman/podman-vs-docker.webp "Docker和Podman的差异")

容器编排技术是目前最重要的Web开发技术之一，而目前在该领域的许多强有力的框架技术正在争夺主导地位。

`Podman`是Red Hat公司的一款产品，以类似`Kubernetes`方式构建、运行和管理容器，其旨在给主流的容器使用人员提供一个可靠的替代。

接下来我们会比较`Podman`与近十年事实上的容器标准`Docker`，因为这两种技术有本质上的区别，但同时也非常适合协同工作。

# 什么是容器编排

容器是包含代码和类库、工具、设置、运行等依赖项的标准软件包，由于它们提供了更快的部署和扩展能力，同时在开发和交付准备阶段能保持一致性，因此业界迅速采用了容器作为容器化架构的核心组件。



容器是轻量级、安全、可移植的，提供了与任何环境兼容的隔离空间。通过将软件从操作系统分离出来，容器可移植到任何地方(如从Linux到Windows)，同时可避免阻碍其正常运行的缺陷和错误。



一些最知名的容器编排技术包括`Docker`、`Docker Swarm`、`Kubernetes`和`Nomad`，我们已经在相关博客中对它们进行了详细的分析和比较。

# 什么是Docker

**`Docker`是标准的容器管理技术**，它在业界具有如此重要的地位，以至于大多数人想到容器时就会想到`Docker`。



`Docker`作为容器编排领域的瑞士军刀，包含了许多其它替代品出现之前的强大功能，它已经成长为一个标准的、自给自足的功能，随着容器管理复杂性的提升，能满足所有开发人员的需求。



`Docker`很快的成为了一个**一体化解决方案**，包含用于特定任务的各种工具，其中一个是`Docker Swarm`，一个原生的`Docker`特性用于集群和调度`Docker`引擎，以及其它的用于创建和管理一大堆容器的工具。



`Docker`的附带工具可用于处理容器编排领域的所有任务，从负载均衡到网络，使得其成为业界的首先，同时也是成熟的参考技术。



但这种自给自足也有问题，尽管`Docker`是软件开发各阶段创建和运行容器的强有力工具，但是其它工具与其交互很困难。近年来随着许多解决特定任务的专用工具出现，`Docker`成为许多专业开发人员将一些操作分配给其它平台和工具的起点。

# 什么是Podman

**那`Podman`又是什么呢? `Podman`是一个基于开放容器标准(Open Container Initiative,OCI)设计的用于开发、管理和运行容器和pod的开源，Linux原生的工具。它是Red Hat开发的用户友好的容器编排器，在RedHat 8和Centos 8中作为默认的容器引擎。**



`Podman`是一系列的命令行工具集合，用于以模块化框架的方式处理容器化过程中的各种不同任务，这些集合包括：

* **Podman**，pod和容器的镜像管理
* **Buildah**，容器构建器
* **Skopeo**，容器镜像检查管理器
* **runc**，`Podman`和`Buildah`使用的容器运行和功能构建器
* **crun**，运行时可选，为无root权限容器提供更大的灵活性、控制与安全性



这些工具也可与任何兼容OCI标准的容器引擎协作，如`Docker`，使得很容易**过渡到`Podman`**或者在已安装的`Docker`环境中使用。那`Kubernetes`能否使用`Podman`？当然可以，事实上它们在某些方面是相似的。



`Podman`对容器有一套不同的概念，

# Podman与Docker的差异对比

`Podman`过去五年的搜索趋势
<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/3728_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Podman","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Podman&hl=en","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

`Docker`过去五年的搜索趋势
<script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Docker","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Docker&hl=en-US","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>


## 架构



## Root权限

## Systemd

## 镜像构建

## Docker Swarm

## 一体化与模块化

# 结论

# 常见问题

