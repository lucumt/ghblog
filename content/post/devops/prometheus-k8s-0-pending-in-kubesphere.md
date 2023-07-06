---
title: "KubeSphere中名为prometheus-k8s-0的pod一直处理Pending状态解决"
date: 2023-06-06T16:17:41+08:00
lastmod: 2023-06-06T16:17:41+08:00
draft: false
keywords: []
description: ""
tags: ["docker","kubernetes","kubesphere"]
categories: ["持续集成","工具使用"]
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

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

简要记录下在安装完成[KubeSphere](https://www.kubesphere.io/)后由于缺少[OpenEBS](https://openebs.io/)而导致名为`prometheus-k8s-0`的pod节点一直处于`Pending`状态，进而导致在`KubeSphere`集群管理中资源统计部分出现NaN的问题。

<!--more-->

1. `KubeSphere`初步安装完成之后，以`admin`账户登录该系统，依次点击`平台管理`->`集群管理`，可发现界面正中的**资源统计**部分很多数据都为**NaN**，影响使用

   ![kubesphere统计结果不正常](/blog_img/devops/prometheus-k8s-0-pending-in-kubesphere/kubesphere-resources-statistics-nan.png "kubesphere统计结果不正常") 

2. 执行`kubectl get po -A`结果如下，可发现名为`prometheus-k8s-0-pod-pending`的pod节点处于`Pending`状态，问题原因初步找到!

   ![名为prometheus-k8s-0的pod处于pending状态](/blog_img/devops/prometheus-k8s-0-pending-in-kubesphere/prometheus-k8s-0-pod-pending.png "名为prometheus-k8s-0的pod处于pending状态") 

3. 执行`kubectl describe pod prometheus-k8s-0 -n kubesphere-monitoring-system`查看该节点信息如下，没有找出特别有用的信息，问题分析暂时陷入僵局

   ![kubectl describe pod执行结果](/blog_img/devops/prometheus-k8s-0-pending-in-kubesphere/kubectl-describe-pod-prometheus-k8s-0.png "kubectl describe pod执行结果") 

4. 在`KubeSphere`的官方论坛找到[这篇文章](https://www.kubesphere.io/forum/d/5445-prometheus-k8s-0-pending)，在其中看到了如下回复，初步判定和`OpenEBS`有关

   ![KubeSphere官方论坛中的相关问题](/blog_img/devops/prometheus-k8s-0-pending-in-kubesphere/kubesphere-forum-related-question.png "KubeSphere官方论坛中的相关问题")

5. 在`GitHub`上找到`OpenEBS`对应的项目，将[openebs-operator.yaml](https://github.com/openebs/openebs/blob/main/k8s/openebs-operator.yaml)下载到对应的服务器，之后执行`kubectl apply -f openebs-operator.yaml `

6. 接着执行`kubectl get po -A`查看对应pod节点，结果如下，可以看出`prometheus-k8s-0`已正常运行

   ![prometheus-k8s-0正常执行](/blog_img/devops/prometheus-k8s-0-pending-in-kubesphere/prometheus-k8s-0-pod-runing.png "prometheus-k8s-0正常执行") 

7. 在`KubeSphere`中刷新页面，显示结果如下，数据可正常显示，至此问题解决!

   ![kubesphere统计结果恢复正常](/blog_img/devops/prometheus-k8s-0-pending-in-kubesphere/kubesphere-resources-statistics-normal.png "kubesphere统计结果恢复正常") 

