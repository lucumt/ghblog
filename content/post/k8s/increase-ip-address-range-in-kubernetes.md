---
title: "给Kubernetes中node的IP地址范围扩容以创建更多的pod"
date: 2023-03-10T22:49:00+08:00
lastmod: 2023-03-10T22:49:00+08:00
draft: true
keywords: ["kubernetes","no IP address available"]
description: "简要介绍通过修改kubesphere中的IP地址范围来实现创建更多的pod"
tags: ["kubernetes","kubesphere"]
categories: ["持续集成","容器化"]
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

由于公司内部越来越多的项目采用`KubeSphere`作为`DevOps`工具，在使用过程中遇到了IP地址耗尽导致pod无法创建的问题，本文简要记录下对应的解决方案。

<!--more-->

## 问题描述

查看发现说是ip地址耗尽

![k8s无可用IP导致无法创建pod](/blog_img/k8s/increase-ip-address-range-in-kubernetes/kubectl-pod-show-no-ip-available.png "k8s由于无可用IP而无法创建pod")   

## 问题排查

一开始已以为是pod数量限制导致的，查看`/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`结果如下

![kubeadm相关配置](/blog_img/k8s/increase-ip-address-range-in-kubernetes/kubeadm-config-output.png "kubeadm相关配置")   

在`KubeSphere`中查看对应的资源用量，发现pod的上限确实已经被调整为500，至此pod数量问题排除。

![kubesphere资源用量展示](/blog_img/k8s/increase-ip-address-range-in-kubernetes/kubesphere-resources-display.png "kubesphere资源用量展示")   

查看`/etc/kubernetes/manifests/kube-controller-manager.yaml `中的相关配置如下

![kube-controller-manager.yaml的输出](/blog_img/k8s/increase-ip-address-range-in-kubernetes/kube-controller-manager-output.png "kube-controller-manager.yaml的输出")   

基于上述文件中配置的IP地址范围，计算出的可用IP地址信息如下，理论上最大能支持**65534**个，按理说不应该这么快耗尽IP地址才对。

![可用IP地址计算结果](/blog_img/k8s/increase-ip-address-range-in-kubernetes/network-available-count-calculate.png "可用IP地址计算结果")   

参考这篇文章[^1]的说明，对node执行类似的操作结果如下，查询出来的CIDR配置为`10.244.0.0/24`，此种配置只能分配254个IP地址，问题原因找到！

![节点cidr查询结果](/blog_img/k8s/increase-ip-address-range-in-kubernetes/node-pod-cidr-output.png "节点cidr查询结果")   

## 解决步骤



[^1]: https://www.cnblogs.com/kdsmyhome/articles/15598327.html

