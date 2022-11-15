---
title: "在Kubernetes中配置service-node-port-range不生效的问题"
date: 2022-08-15T18:27:59+08:00
lastmod: 2022-08-15T18:27:59+08:00
draft: false
keywords: ["kubernetes","service-node-port-range"]
description: "简要自己在kubernetes中配置service-node-port-range不生效问题的解决方案"
tags: ["kubernetes"]
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

近期在公司内部搭建基于[**KubeSphere**](https://kubesphere.com.cn/)的持续集成平台时，发现其底层的[**Kubernetes**](https://kubernetes.io/)默认的端口范围为**30000-32767**而公司有多个采用微服务模块的项目在使用，默认的端口范围不便于分配使用，在基于网上文档修改的过程中自己踩到了一个坑，简单记录下。

<!--more-->

一开始时自己也是从网上查找相关资料，查找出来的结果和[**修改NodePort的范围**](https://kuboard.cn/install/install-node-port-range.html)中的基本都相同，自己也是按照其对应的方法进行操作:

1. 将`/etc/kubernetes/manifests/`目录下的`kube-apiserver.yaml`在当前目录下备份一份`kube-apiserver.yaml.bak`作为备份文件

2. 在`kube-apiserver.yaml`中加入`--service-node-port-range=1-65537`

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     creationTimestamp: null
     labels:
       component: kube-apiserver
       tier: control-plane
     name: kube-apiserver
     namespace: kube-system
   spec:
     containers:
     - command:
       - kube-apiserver
       - --service-node-port-range=1-65537
       # other config
   ```

3. 之后执行下述命令来重启api-server

   ```bash
   # 获得 apiserver 的 pod 名字
   export apiserver_pods=$(kubectl get pods --selector=component=kube-apiserver -n kube-system --output=jsonpath={.items..metadata.name})
   # 删除 apiserver 的 pod
   kubectl delete pod $apiserver_pods -n kube-system
   ```

4. 然后执行`kubectl describe pod $apiserver_pods -n kube-system | grep "service-node-port-range"`却发现输出结果为空，也就是说自己添加的动态端口配置并没有生效！

   ![service-node-port-range不生效](/blog_img/k8s/service-node-port-range-config-not-working-in-k8s/service-node-port-range-not-find.png "service-node-port-range不生效") 

5. 之后搜索网上的各种解决方案，都是和前述的操作类似，尝试后都不生效，没办法只能继续搜索，直到发现[https://www.modb.pro/db/146676](https://www.modb.pro/db/146676)这篇文章，在其中有如下说明

   ![service-node-port-range不生效排查](/blog_img/k8s/service-node-port-range-config-not-working-in-k8s/service-node-port-range-not-find-analysis.png "service-node-port-range不生效排查") 

6. 之后检查自己系统的`/etc/kubernetes/manifests/`目录，发现果然有备份文件!

   ![备份文件存在](/blog_img/k8s/service-node-port-range-config-not-working-in-k8s/kube-manifests-file-list.png "备份文件存在") 

7. 将备份文件删除后，重新执行`kubectl delete pod $apiserver_pods -n kube-system`，经过若干秒的等待后，发现`service-node-port-range`已经生效，至此问题解决!

   ![service-node-port-range生效](/blog_img/k8s/service-node-port-range-config-not-working-in-k8s/service-node-port-range-working.png "service-node-port-range生效") 

总结：

1. 虽然这次问题是由于备份文件引起的，但是在`Linux`中修改重要配置文件时提前进行备份是个好习惯，从这次修改过程中学到的经验是**不要在同目录下备份，要去专门的目录下备份**。
2. `Kubernetes`底层的动态加载配置文件的原理需要进一步学习！
3. 感谢[https://www.modb.pro/db/146676](https://www.modb.pro/db/146676)的作者[埋头过坎](https://www.modb.pro/u/447187)，由于其无私分享避免了我走更多的弯路。

