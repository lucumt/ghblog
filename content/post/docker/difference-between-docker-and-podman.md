---
title: "[译]Docker和Podman的差异"
date: 2024-04-26T10:46:34+08:00
lastmod: 2024-04-26T10:46:34+08:00
draft: false
keywords: ["Docker","Podman","容器化","容器编排","差异"]
description: "翻译一篇关于Docker和Podman的差异的博文,以便技术选型时能提供参考"
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

## 什么是容器编排

容器是包含代码和类库、工具、设置、运行等依赖项的标准软件包，由于它们提供了更快的部署和扩展能力，同时在开发和交付准备阶段能保持一致性，因此业界迅速采用了容器作为容器化架构的核心组件。



容器是轻量级、安全、可移植的，提供了与任何环境兼容的隔离空间。通过将软件从操作系统分离出来，容器可移植到任何地方(如从Linux到Windows)，同时可避免阻碍其正常运行的缺陷和错误。



一些最知名的容器编排技术包括`Docker`、`Docker Swarm`、`Kubernetes`和`Nomad`，我们已经在相关博客中对它们进行了详细的分析和比较。

## 什么是Docker

**`Docker`是标准的容器管理技术**，它在业界具有如此重要的地位，以至于大多数人想到容器时就会想到`Docker`。



`Docker`作为容器编排领域的瑞士军刀，包含了许多其它替代品出现之前的强大功能，它已经成长为一个标准的、自给自足的功能，随着容器管理复杂性的提升，能满足所有开发人员的需求。



`Docker`很快的成为了一个**一体化解决方案**，包含用于特定任务的各种工具，其中一个是`Docker Swarm`，一个原生的`Docker`特性用于集群和调度`Docker`引擎，以及其它的用于创建和管理一大堆容器的工具。



`Docker`的附带工具可用于处理容器编排领域的所有任务，从负载均衡到网络，使得其成为业界的首先，同时也是成熟的参考技术。



不过这种自给自足也有问题，尽管`Docker`是软件开发各阶段创建和运行容器的强有力工具，但其它工具与其交互很困难。近年来随着许多解决特定任务的专用工具出现，`Docker`成为许多专业开发人员将一些操作转移到其它平台和工具的分水岭。

## 什么是Podman

**那`Podman`又是什么呢? `Podman`是一个基于开放容器标准(Open Container Initiative,OCI)设计的用于开发、管理和运行容器和pod的开源，Linux原生的工具。它是Red Hat开发的用户友好的容器编排器，在RedHat 8和Centos 8中作为默认的容器引擎。**



`Podman`是一系列的命令行工具集合，用于以模块化框架的方式处理容器化过程中的各种不同任务，这些集合包括：

* **Podman**，pod和容器的镜像管理
* **Buildah**，容器构建器
* **Skopeo**，容器镜像检查管理器
* **runc**，`Podman`和`Buildah`使用的容器运行和功能构建器
* **crun**，运行时可选，为无root权限容器提供更大的灵活性、控制与安全性



这些工具也可与任何兼容OCI标准的容器引擎协作，如`Docker`，使得很容易 **过渡到`Podman`** 或者在已安装的`Docker`环境中使用。那`Kubernetes`能否使用`Podman`？当然可以，事实上它们在某些方面是相似的。



`Podman`对容器有一套不同的概念，正如其名称所暗示的那样，`Podman`可以创建协同工作的容器pods，**类似于`Kubernetes`中的pod功能**。Pods将多个不同的容器组合到一个共同的名称下以单个单元的形式来管理它们。



这样做的好处是开发人员可以共享资源，在pod内为同一个应用程序使用不同的容器：一个用于前端，一个用于后端，另一个用于数据库。**Pod的定义可以导出为兼容`Kubernetes`的yaml文件**并运行用于`Kubernetes`集群中，使得容器能够更快的投入实际使用。



**`Podman`的另一个显示特征是其没有守护进程**，守护进程是在后台运行的程序，用于处理没有用户界面的服务、进程和请求。其对容器引擎进行了独特的改进，因为它实际上并不依赖于守护进程，而是将容器和 Pod 作为子进程启动。



或许你会问 **为啥我要用`Podman`?** 这是由于它作为开发和管理工具具有独特的优势，使其成为在适当的环境下成为可行且有趣的`Docker`替代品，也可作为与`Docker`协同工作的强有力补充，因为它支持与`Docker`兼容的CLI标准接口。

## Podman与Docker的差异对比

根据 Google Trends 的数据，过去五年`Docker`和`Podman`的关注度一直在波动，但`Docker`相对更受欢迎，截止到目前，这两个容器编排工具都获得了用户极大的关注度。



`Podman`过去五年的搜索趋势

<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/3728_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Podman","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Podman&hl=en","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

`Docker`过去五年的搜索趋势
<script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Docker","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Docker&hl=en-US","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

`Podman`和`Docker`有许多共同的特性，但也存在一些根本上的区别，这并不意味着一个比另一个好，而是要根据具体项目做出合适的选择。

### 架构

**`Docker`使用守护线程**，一个后台持续运行的程序来创建镜像和容器，`Podman`则使用无守护线程的架构，这意味着它可以在启动容器的用户下运行容器，`Docker`拥有由守护进程管控的客户端-服务器交互逻辑，另外一个则没有。

### Root权限

`Podman`由于不需要守护进程来管理其活动，同样不需要对容器分配root权限，`Docker`最近在其守护进程配置中添加了无root模式，但`Podman`首先使用了这种方法，并将其推广为一项基本特性，接下来会具体说明。

### 安全

**`Podman`是否比`Docker`更安全**？`Podman`允许容易以非root的方式运行，相对于具有root权限的容器而言无root容器通常更安全。在`Docker`中守护线程拥有root权限，使得其成为攻击者的理想入口。`Podman`中的容器默认情况下没有root权限，从而在root和非root级别添加了一个天然的屏障来提高安全性，同时，它也能同时运行有root权限和无root权限的容器。

### Systemd

由于没有守护线程，`Podman`需要其它的工具在后台来管理服务和支持容器运行，Systemd可用于为已有容器创建控制单元或创建新容器，Systemd还可以与`Podman`集成，允许它运行默认启用基于Systemd的容器且无需任何修改。



通过使用Systemd，供应商可以以容器的形式安装、运行和管理他们的应用程序，因为现在大多数应用程序都是以这种方式打包和交付的。

### 镜像构建

作为一个自给自足的工具，`Docker`可自行构建容器镜像，而`Podman`则需要一个名为`Buildah`工具的协助，这正好表明了`Podman`的一个特点：用于运行容器而不是用于构建容器。

### Docker Swarm

`Podman`不支持Docker Swarm,由于使用Docker Swarm会产生操作，这会将其排除在使用Docker Swarm特定的项目之外。`Podman`近期添加了Docker Compose的支持，使其与Swarm兼容，从而解决这一限制。`Docker`则很自然的可以与 Swarm良好配合使用。

### 一体化与模块化

这是这两个工具的另外一个关键差异：`Docker`是一个体式、功能强大且独立的工具，包含各种优点和缺点，可处理整个周期内的所有容器化任务，`Podman` 则采用模块化的方式，对于特定的任务依赖特定的工具来实现。

## 结论

`Podman`和`Docker`都是功能强大的容器编排工具，各有优势和区别，尽管`Docker`已成为近十年的容器行业标准，但`Podman`的创新架构和容器管理方法使其成为开发人员（尤其是在Linux环境中工作的开发人员）的可靠选择。



不论你选择哪一种，抑或是两者都用，了解它们的差别和共同点有助于对项目做出最佳的决定。

## 常见问题

### Podman能否替代Docker

是的，`Podman`可以在许多场景下替换`Docker`，它提供了与`Docker`类似的容器运行时环境和工具，并且在某种程度上，能提供额外好处，如更高的安全性和灵活性。

### Podman与Docker有何不同

`Podman`相对于`Docker`的一个主要区别是它不需要单独的守护进程，使得其更轻量级和更安全，同样也为非root容器运行提供了更好的支持以提升安全。除此之外，`Podman`可在不需要类似Docker Compose这种工具的支持下原生运行`Kubernetes`中的pods。



想了解更多吗？可以按照`Docker`+`Kubernetes`去查找。

### Podman是否比Docker更安全

`Podman`有时被认为比`Docker`更安全，这是由于它不需要单独的守护进程来运行容器，从而减少了潜在安全漏洞的攻击面，同时还更好地支持以非root 用户身份运行容器，从而可以提高安全性。

### 哪种更合适：Podman还是Docker

`Podman 和 Docker`哪个更好呢？`Podman`是否比`Docker`更好取决于具体的场景和需求。有时`Podman`可能提供更好的安全性和灵活性，但`Docker`可能更适合某些环境或应用程序。这两个考量点很重要，因为可用于确定哪个最能满足项目的需求。