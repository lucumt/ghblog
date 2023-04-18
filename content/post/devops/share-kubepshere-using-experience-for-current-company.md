---
title: "KubeSphere在当前公司提升研发效率的使用实践分享"
date: 2023-04-17T14:31:45+08:00
lastmod: 2023-04-17T14:31:45+08:00
draft: true
tags: ["kubesphere","kubernetes","docker","jenkins","nacos","shell"]
description: "KubeSphere在当前公司提升研发效率的使用实践分享"
tags: ["kubesphere","kubernetes","docker","jenkins","nacos","shell"]
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

我司从2022年6月开始使用`KubeSphere`到目前为止快一年时间，简要记录下此过程中的经验积累和问题反馈。

<!--more-->

# 背景

公司当前有接近3000人的规模，主要业务为汽车配套相关的软硬件开发，其中专门从事软件开发约有800人，这其中`Java`开发的约占70%，余下的为`C/C++`嵌入式和`C#`桌面程序的开发。

在`Java`开发部分约80%的都是`Java EE`开发，由于公司的业务主要是给外部客户提供软硬件产品和咨询服务，在早期公司和部门更关注的是如何将产品销售给更多的客户、获得更多的订单和尽快回款，对软件开发流程这块没有过多的重视，故早期在软件开发部分不是特别规范化。软件开发基于项目主要采用[敏捷开发](https://zh.wikipedia.org/zh-hans/%E6%95%8F%E6%8D%B7%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91)或[瀑布模型](https://zh.wikipedia.org/wiki/%E7%80%91%E5%B8%83%E6%A8%A1%E5%9E%8B)，而对于软件部署和运维依旧采用的纯手工方式(PS:请不要鄙视我司!)

随着公司规模的扩大与软件产品线的增多，上述方式逐渐暴露出一些问题：

* 存在大量重复性工作，在软件快速迭代时，需要频繁的手工编译部署，耗费时间，且此过程缺乏日志记录，后续无法追踪审计
* 缺乏审核功能，对于测试环境和生产环境的操作需要审批流程，之前通过邮件和企业微信无法串联
* 缺乏准入功能，随着团队规模扩大，人员素质参差不齐，需要对软件开发流程、代码风格都需要强制固化
* 缺乏监控功能，后续不同团队、项目采用的监控方案不统一，不利于知识的积累
* 不同客户的定制化功能太多(logo,字体，ip地址，业务逻辑等)，采用手工打包的方式效率低，容易遗漏

基于上述原因，部门准备选用网络上开源的系统来尽可能的解决上述痛点，在技术选型时有如下考量点：

* 采用尽量少的系统，最好一套系统能解决前述所有问题，避免多个系统维护和整合的成本
* 安装过程简洁，不需要复杂的操作，能支持离线安装
* 文档丰富、社区活跃、使用人员较多，遇到问题能较容易的找到答案
* 受限于部门预算，尽量使用开源版本，使用成熟后可考虑商业版本

# 实践过程

![不同类型的构建方式比较](/blog_img/devops/share-kubepshere-using-experience-for-current-company/different-type-of-deploy-compare.png "不同类型的构建方式比较") 

# 使用效果



# 规划&反馈

## 规划

结合公司与部门的实际情况，短期的规划依然是完善基于`Jenkins`的`CI`/`CD`使用来完善打包与部署流程，部门内部在进行全面`web`化，基于此中长期拥抱云原生。

* 接入企业微信，将构建与运行结果随时通知相关人
* 将部门内部基于`Eclipse RCP`的桌面应用程序通过`Jenkins`实现标准化与自动化的构建
* 部门内部的`web`项目全部通过`KubeSphere`构建部署，完善其使用文档，挖掘`KubeSphere`在部门业务中新的应用场景
* 将底层的`Kubernetes`从单机升级为集群，支持更多`pod`的部署，支持公司内部需要大量`pod`并发运行的云仿真项目

## 问题反馈

1. **对离线操作提供更多支持**，同很多其它公司一样，我司的研发主要在公司内部网络进行，对于互联网的连接受限制，只开放了对于`docker`，`maven`等少数仓库的代理镜像访问，希望能对离线安装与升级提供更多的支持，类似`kubekey`这种，在内部网络简单配置镜像地址后即可快速安装

2. **使用体验优化**,基于公司内部使用过程中大家反馈的一些问题：

   * 重复执行流水线时，显示的用户为`admin`而非当前实际登录的用户

     ![重复执行时用户名显示为admin](/blog_img/devops/share-kubepshere-using-experience-for-current-company/rerun-username-displays-as-admin.png "重复执行时用户名显示为admin") 

   * 制品文件通过下载按钮无法下载，只能通过文件名下载

     ![制品文件下载出错](/blog_img/devops/share-kubepshere-using-experience-for-current-company/archive-download-error-occurs.png "制品文件下载出错") 

   * 之前从`3.1`升级到`3.3` 版本的一个主要原因是`3.1`的权限控制不完善，但在`3.3`时发现当把用户角色设置为非`admin`时，导致相关用户无法看见终端，部分场景下分析调试不方便

     ![非admin角色无法看见终端](/blog_img/devops/share-kubepshere-using-experience-for-current-company/no-admin-role-can-not-see-terminal.png "非admin角色无法看见终端") 

   * 公司网络环境多变且随着使用项目的增多，经常需要备份/迁移，目前的方式是手工备份`Jenkins`文件，但在`Jenkins`流水线构建参数无法体现在`Jenkins`流水线中，每次备份/迁移后都需要手工创建参数

     ![有多个构建参数的流水线](/blog_img/devops/share-kubepshere-using-experience-for-current-company/jenkins-pipeline-with-multiple-parameters.png "有多个构建参数的流水线") 

# 参考文章

* [KubeSphere使用心得](/post/devops/share-experiences-for-using-kubesphere/)
* [Kubesphere集成LDAP踩坑记录](/post/devops/setting-ldap-for-kubesphere/)
* [利用Nacos与KubeSphere创建多套开发与测试环境](/post/devops/using-nacos-and-kubesphere-to-create-multiple-environments/)
* [在Jenkins中根据配置从不同的仓库中Checkout代码](/post/devops/git-checkout-by-dynamic-repository-in-jenkins/)
* [由于k8s中的错误配置导致无法创建新节点](/post/k8s/cpu-resource-not-enough-due-to-invalid-k8s-config/)