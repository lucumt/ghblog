---
title: "由于k8s中的错误配置导致无法创建新节点"
date: 2022-11-11T18:19:12+08:00
lastmod: 2022-11-11T18:19:12+08:00
draft: false
keywords: ["kubernetes","kubesphere"]
description: "简要介绍自己在使用kubesphere过程中由于错误的配置导致cpu资源紧张无法创建新节点的问题"
tags: ["kubernetes","kubesphere","linux"]
categories: ["持续集成","容器化"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
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

记录下自己在使用[KubeSphere](https://kubesphere.io/)过程中由于`YAML`中错误的配置导致`Kubernetes`经常资源紧张而无法创建与部署新节点的问题。

<!--more-->

## 问题描述

部门在使用`KubeSphere`过程中经常遇到类似如下图所示的由于`kubernetes`无法调度而导致的无法部署的问题

![kubesphere部署失败](/blog_img/k8s/cpu-resource-not-enough-due-to-invalid-k8s-config/kubesphere-can-not-deploy-error.png "kubesphere部署失败") 

进一步查看报错节点信息会发现提示CPU资源不足，从而导致无法部署。

![内存不足导致k8s无法调度](/blog_img/k8s/cpu-resource-not-enough-due-to-invalid-k8s-config/kubesphere-insufficient-cpu.png "内存不足导致k8s无法调度") 

之前自己采取的解决方案是暴力的删除一部分节点来释放CPU资源，但此种方式治标不治本，且随着`KubeSphere`在部门内部使用的普及，此问题发生的频率越来越高，只能想办法从根源上处理。

## 分析与解决

由于提示的是CPU资源不足，首先检查是否为CPU的问题，采用`lscpu | egrep 'Model name|Socket|Thread|NUMA|CPU\(s\)'`和`free -g`分别查看CPU和内存信息，结果如下

![服务器CPU和内存信息](/blog_img/k8s/cpu-resource-not-enough-due-to-invalid-k8s-config/linux-server-cpu-mem-info.png "服务器CPU和内存信息") 

从查询结果可知系统的CPU配置和内存配置都很高，而自己在`KubeSphere`中部署的都是一些普通的Java程序，不可能占用特别多的CPU和内存资源，Linux服务器本身的配置问题排除。

接下来利用`kubectl describe node`查看节点信息，输出结果如下

![查看k8s节点信息](/blog_img/k8s/cpu-resource-not-enough-due-to-invalid-k8s-config/kubectl-describe-node.png "查看k8s节点信息") 

在上图中可发现相关节点汇总后的CPU和内存占用的百分比非常大，其中CPU在Requests部分占比为84%，由于Linux系统自身运行和运行`Kubesphere`与`Kubernetes`都需要占用一定的CPU资源，从而导致当`Kubersphere`中部署的项目超过一定数量时，Linux系统无法给`Kubernetes`分配足够的CPU资源导致部署失败。至此，问题的表面原因找出来了。

进一步分析上面的问题，自己觉得很好奇的是为啥Requests部分占用的资源，自己在对应的`YAML`文件中压根就没指定Requests相关的参数，检查代码发现Limits配置的值比较大

```yaml
resources: 
  limits: 
    cpu: 500m
    memory: 2000Mi
```

同时基于`kubectl describe node`发现相关节点的Requests和Limits的值都相同，猜测是否`Kubernetes`默认将Ruquests的值设置为与Limits的值相同。

![k8s节点limits和requests相同](/blog_img/k8s/cpu-resource-not-enough-due-to-invalid-k8s-config/kubectl-nodes-requests-and-limits.png "k8s节点limits和requests相同") 

在[Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)官网发现如下说明

> **Note:** If you specify a limit for a resource, but do not specify any request, and no admission-time mechanism has applied a default request for that resource, then Kubernetes copies the limit you specified and uses it as the requested value for the resource.

上述文字说明当我们在`YAML`文件中只设置了Limits但是没有指定Requests，则`Kubernetes`会将Request的值默认设置为与`Limits`相同，从而导致CPU和内存资源占用都很高，至此问题根源找到！

解决的方式也很简单，要么移除掉Limits，让`Kubernetes`根据Linux系统资源和节点状况给我们动态的分配节点，或者同时指定Requests和Limits即可。

```yaml
# 屏蔽此部分代码
#resources: 
#  limits: 
#    cpu: 500m
#    memory: 2000Mi 

# 或者显示配置request和limits
resources: 
  limits: 
    cpu: 500m
    memory: 2000Mi
  requests: 
    cpu: 20m
    memory: 64Mi
```

经验教训： 需要对自己项目中的代码要有更深入的理解，尤其是配置文件，要做到理解每一行的作用，不能简单的复制别人的代码。

----

参考文档：

* https://kubesphere.io/zh/blogs/deep-dive-into-the-k8s-request-and-limit/

* https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/