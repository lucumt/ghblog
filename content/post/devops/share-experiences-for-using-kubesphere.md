---
title: "KubeSphere使用心得"
date: 2022-08-06T18:00:08+08:00
lastmod: 2022-08-06T18:00:08+08:00
draft: true
keywords: ["kubesphere","Jenkins","持续集成","持续部署"]
description: "KubeSphere使用心得分享"
tags: ["kubesphere","jenkins"]
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

目前公司的开发方式都是手工编译&部署，十分低效，最近将Web开发相关的项目都基于[KubeSphere](https://kubesphere.io/)通过基于[Jenkins](https://www.jenkins.io/)的流水线方式实现了自动部署，在此过程中遇到了一些阻塞点，简单记录下它们的解决方案。

<!--more-->

在`KubeSphere`的[Github仓库](https://github.com/kubesphere/kubesphere/blob/master/README_zh.md)中有如下说明：

{{% admonition info "KubeSphere是什么"  %}} 

[KubeSphere](https://kubesphere.io/zh/) 愿景是打造一个以 [Kubernetes](https://kubernetes.io/zh/) 为内核的 **云原生分布式操作系统**，它的架构可以非常方便地使第三方应用与云原生生态组件进行即插即用（plug-and-play）的集成，支持云原生应用在多云与多集群的统一分发和运维管理。 KubeSphere 也是一个多租户容器平台，提供全栈的 IT 自动化运维的能力，简化企业的 DevOps 工作流

 {{% /admonition %}}

{{% admonition info "Kubernetes DevOps" %}} 

提供开箱即用的基于 Jenkins 的 CI/CD，并内置自动化流水线插件，包括 Binary-to-Image (B2I) 和 Source-to-Image (S2I)

 {{% /admonition %}}

从上述描述可知`KubeSphere`的两大底层支柱为`Kubernetes`与`Jenkins`，本文也主要基于这两部分进行记录。

# 流水线设计

基于部门现状以及参考网上资料，将产品部署环境划分为如下4种类型:

* **dev**，本地开发阶段
* **sit**，前后端各模块联调阶段
* **test**，软件功能测试阶段
* **prod**，正式使用阶段

虽然通过上述划分方式可有效的避免不同环境的互相干扰，但由于目前部门大部分产品都采用了[微服务](https://zh.wikipedia.org/zh-cn/%E5%BE%AE%E6%9C%8D%E5%8B%99)实现，软件架构类似下图所示，每个微服务模块都有单独的代码管理，导致若给前后端的每个功能模块在不同环境下都配置一套[Jenkins流水线](https://www.jenkins.io/zh/doc/book/pipeline/)则实际的流水线数目会十分庞大，不便于管理和维护。

![项目架构图](/blog_img/devops/share-experiences-for-using-kubesphere/project-architecture.png "项目架构图")  

若对于每个功能模块在所有不同环境中都分别使用一套流水线，则其数量为**功能模块数X环境种类数**，造成流水线数目过大，使用不便同时也不利于后期的维护。

基于此，我们对`Jenkins`流水线的构建和使用方式提出了如下要求：

* 一套`Gitlab`代码库对应一条`Jenkins`流水线，实际上就是前后端的一个功能模块，使用时可动态选择分支
* 一套`Jenkins`流水线可以根据使用需求灵活的往`dev`、`sit`、`test`、`prod`这4套环境之一进行部署
* `dev`、`sit`、`test`、`prod`对于同一套代码而言是分别部署的，即4套环境互不影响

# 动态参数设置

![Jenkins构建流程](/blog_img/devops/share-experiences-for-using-kubesphere/jenkins-build-flow.png "Jenkins构建流程")  

上图展示了`Jenkins`进行软件构建的主要流程，从中可知若达到上述精简流水线条目的要求，则需要进行**动态参数配置**，基于项目实际情况，整理出如下场景：

1. **软件版本**，主要是后端基于[Maven](https://maven.apache.org/)的项目需要正确的获取版本号
2. **容器端口**，`SpringBoot`应用程序对外暴露的端口，此部分需要在使用`Dockerfile`构建时动态对外暴露[^1]
3. **容器端口**，`Kubernetes`中创建容器时对外暴露的端口，主要用于程序访问 
4. **镜像版本**，前后端程序构建镜像时，对应tag的动态设置

## shell脚本

## scripti脚本

[script console](https://www.jenkins.io/doc/book/managing/script-console/)

# 代码编译

由于公司研发环境与互联网隔离，故需要在前后端分别设置可用的镜像才能确保编译正常。

## 前端

由于`KubeSphere`中的[Node.js](https://nodejs.org/en/)的版本落后于项目中使用的版本，故需要通过如下命令升级其版本：

`npm install node@16.13.1 --registry https://mirrors.xxx.com/repository/NPM/`

## 后端

* 获取`Maven`版本号：

  `mvn help:evaluate -Dexpression=project.version -q -DforceStdout`

* Script脚本：

  ```groovy
  env.PROJECT_VERSION = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true)
  env.BUILD_TIME = new Date().format(“yyyyMMdd-HHmmss“)
  env.BUILD_TAG = PROJECT_VERSION + “-“ + BUILD_TIME
  ```


# K8S容器探活

`KubeSphere`对于容器的启动、就绪和存活的探测依赖于`Kubernetes`的[相关实现](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)，实际使用中`KubeSphere`只需要能正常检测到就绪状态即可，现阶段项目中也只基于就绪探测来实现。

## 后端

`SpringBoot`程序可基于[Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/production-ready.html)中提供的`actuator/health`进行就绪检测，相关配置如下:

```yaml
containers:
  - image: '$REGISTRY/$DOCKERHUB_NAMESPACE/idp-system:${BUILD_TAG}'
    readinessProbe:
      httpGet:
        path: idp-system/actuator/health
        port: $NODE_PORT
      timeoutSeconds: 10
      failureThreshold: 30
      periodSeconds: 5
```

## 前端

由于前端没有单独的接口对外暴露，故可采用在`Docker`容器中执行linux命令的方式来检测是否就绪，个人采用`uname`，相关配置如下：

```yaml
containers:
  - image: '$REGISTRY/$DOCKERHUB_NAMESPACE/idp-system-web:${BUILD_TAG}'
    readinessProbe:
      exec:
        command:
          - uname
      timeoutSeconds: 10
      failureThreshold: 30
      periodSeconds: 5
```

# 参考代码

* 前端部分:
  * **Dockerfile**：
  * **depoly.yml**：
  * **Jenkins流水线**，参见[idp-common-web.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/idp-common-web.groovy "点击跳转到对应的Github链接")

* 后端部分:
  * **Dockerfile**：
  * **depoly.yml**：
  * **Jenkins流水线**，参见[idp-system.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/idp-system.groovy "点击跳转到对应的Github链接")

参考:

1. https://askubuntu.com/questions/76808/how-do-i-use-variables-in-a-sed-command

[^1]: 理论上来说此处的端口不用暴露，只需要在容器创建时进行对应的端口映射即可，不过由于公司项目采用[nacos](https://nacos.io/zh-cn/)进行动态配置与服务发现，为了方便统一管理，故也将此处的端口做成动态配置