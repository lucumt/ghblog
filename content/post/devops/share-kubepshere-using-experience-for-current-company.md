---
title: "KubeSphere在当前公司提升研发效率的使用实践分享"
date: 2023-04-17T14:31:45+08:00
lastmod: 2023-04-17T14:31:45+08:00
draft: false
tags: ["kubesphere","kubernetes","docker","jenkins","nacos","shell"]
description: "KubeSphere在当前公司提升研发效率的使用实践分享"
tags: ["kubesphere","kubernetes","docker","jenkins","nacos","shell"]
categories: ["持续集成"]
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

在竞争日益激烈的市场环境下，**公司需要把有限的人力资源优先用于业务迭代开发**，解决上述问题变得愈发迫切。

# 选型说明

基于前述原因，部门准备选用网络上开源的系统来尽可能的解决上述痛点，在技术选型时有如下考量点：

* 采用尽量少的系统，最好一套系统能解决前述所有问题，避免多个系统维护和整合的成本
* 采用开源版本，避免公司内部手工开发，节约人力
* 安装过程简洁，不需要复杂的操作，能支持离线安装
* 文档丰富、社区活跃、使用人员较多，遇到问题能较容易的找到答案
* 支持容器化部署，公司和部门的业务中自动驾驶和云仿真相关的越来越多，此部分对算力和资源提出了更高的要求

最开始采用的是[Jenkins](https://www.jenkins.io/)，通过`Jenkins`基本上能解决我们90%的问题，但依旧有如下问题影使用体验:

* 对于云原生支持不太好，不利于部门后续云仿真相关的业务使用
* UI界面简陋，交互方式不友好(项目构建日志输出等)
* 对于项目，资源的权限分配与隔离过于简陋，不满足多项目多部门使用时细粒度的区分要求

在网络上查找后发现类似的工具有很多，经过初步对比筛选后倾向于`KubeSphere`、[Walle](https://github.com/meolu/walle-web)，[Zadig](https://koderover.com/)这2款产品，它们的基本功能都类似，进一步对比如下：

|                  | KubeSphere |        Zadig         |
| ---------------- | :--------: | :------------------: |
| **云原生支持**   |   **高**   |         一般         |
| **UI美观度**     |   **高**   |         一般         |
| **GitHub Star**  | **12.4k**  |          2k          |
| **社区活跃度**   |   **高**   |         一般         |
| **知名使用客户** |  去哪儿网  | **飞书、腾讯、华为** |

`KubeSphere`在多个指标上优于`Zadig`，但在使用客户上没有像`Zadig`这么多知名的客户，不过我们的第一考量点还是能否满足自身要求，结合上述对比指标，尤其是`KubeSphere`的UI界面十分美观比`Zadig`强很多，故最终选定`KubeSphere`作为部门内部的持续集成与容器化管理系统！

至此，部门内部经历了`手工操作`->`Jenkins`->`KubeSphere`这3个阶段，各阶段的主要使用点如下：

![不同类型的构建方式比较](/blog_img/devops/share-kubepshere-using-experience-for-current-company/different-type-of-deploy-compare.png "不同类型的构建方式比较") 

# 实践过程

`KubeSphere`在公司内部的整体部署架构如下图所示，其作为最顶层的应用程序直接与使用人员交互，提供主动 /定时触发构建、应用监控等功能，使用人员不必关心底层的`Jenkins`、`Kubernetes`等依赖组件。s

![kubesphere整合架构图](/blog_img/devops/share-kubepshere-using-experience-for-current-company/kubesphere-integration-architecture.png "kubesphere整合架构图") 

## 持续集成

## 应用监控

## 外部部署

部门内部的软件最终都会销售并交付给相关客户，由于客户网络与公司网络不通以及代码保密等要求，无法在客户现场使用原有的`Jenkins`流水线进行部署交付。基于此部门采取折中方案：**在公司内部通过`KubeSphere`进行编译打包，导出`docker`镜像，拷贝到客户处然后基于`docker`镜像部署运行**,具体请参见如下链接:

* [在Jenkins中根据配置从不同的仓库中Checkout代码](/post/devops/git-checkout-by-dynamic-repository-in-jenkins/)
* [利用shell脚本实现将微服务程序以docker容器方式自动部署](/post/linux/using-script-to-deploy-microservice-application-in-docker/)

![客户环境部署流程图](/blog_img/devops/share-kubepshere-using-experience-for-current-company/software-deploy-in-customer-environment.png "客户环境部署流程图") 

## 使用协助

在使用过程中确实遇到了不少问题，主要通过如下三条途径解决:

* 阅读[官方文档](https://kubesphere.io/zh/docs/v3.3)，根据文档说明操作
* 若官网文档没有，则去[用户论坛](https://kubesphere.io/forum/)查看是否有人遇到类似问题或直接发帖
* 通过微信群寻求协助

根据部门经验，90%的问题可通过官方文档或用户论坛解决。

# 使用效果

部分同事习惯于原始的手工操作或基于`docker`部署，导致在推广过程中受到了一定的阻力，部门内部基于充分沟通和逐步替换的方式引导相关同事来慢慢适应。经过约一年的时间磨合，大家都认可了拥抱云原生和`KubeSphere`给我们带来的便利。

对我司而言，有如下几个方面的提升:

* 研发人员几乎不用耗费时间在软件的部署和监控上，节省约20%时间，产品迭代速度更快
* 定制化的功能通过脚本实现，彻底杜绝了给客户交付软件时由于人工疏漏导致的偶发问题，在提高软件交付质量的同时也提升了客户我司的认可度
* 软件开发、测试流程更规范，通过在`Jenkins`流水线强制添加各种规范检查和审核流程，实现了软件研发的规范统一，代码质量更高，更利于扩展维护，同时也在一定程序上减少了由于人员流失/变更对项目造成的影响
* 基于`KubeSphere`的云原生部署结合`Nacos`可以更快速的分配多套环境，有效的实现了`开发`、`测试`、`生产`环境的隔离，在云仿真相关的业务场景中可基于业务场景更方便的对`pod`进行监控与调整，前瞻性的业务研发开展更顺利

![日常开发中采用KubeSphere进行集成](/blog_img/devops/share-kubepshere-using-experience-for-current-company/using-kubesphere-to-deploy.png "日常开发中采用KubeSphere进行集成") 

# 规划&反馈

## 规划

结合公司与部门的实际情况，短期的规划依然是完善基于`Jenkins`的`CI`/`CD`使用来完善打包与部署流程，部门内部在进行全面`web`化，基于此中长期拥抱云原生。

* 接入企业微信，将构建与运行结果随时通知相关人，构建结果与项目监控更实时
* 将部门内部基于`Eclipse RCP`的桌面应用程序通过`Jenkins`实现标准化与自动化的构建
* 将底层的`Kubernetes`从单机升级为集群，支持更多`pod`的部署，支持公司内部需要大量`pod`并发运行的云仿真项目
* 部门内部的`web`项目全部通过`KubeSphere`构建部署，完善其使用文档，挖掘`KubeSphere`在部门业务中新的应用场景(如对设计文档、开发文档、bug修复的定时与强制检查通知等)

## 问题反馈

1. **对离线操作提供更多支持**，同很多其它公司一样，我司的研发主要在公司内部网络进行，对于互联网的连接受限制，只开放了对于`docker`，`maven`等少数仓库的代理镜像访问，希望能对离线安装与升级提供更多的支持，类似`kubekey`这种，在内部网络简单配置镜像地址后即可快速安装

2. **使用体验优化**,基于公司内部使用过程中大家反馈的一些问题：

   * 重复执行流水线时，显示的用户为`admin`而非当前实际登录的用户

     ![重复执行时用户名显示为admin](/blog_img/devops/share-kubepshere-using-experience-for-current-company/rerun-username-displays-as-admin.png "重复执行时用户名显示为admin") 

   * 制品文件通过下载按钮无法下载，只能通过文件名下载

     ![制品文件下载出错](/blog_img/devops/share-kubepshere-using-experience-for-current-company/archive-download-error-occurs.png "制品文件下载出错") 

   * 之前从`3.1`升级到`3.3` 版本的一个主要原因是`3.1`的权限控制不完善，但在`3.3`时发现当把用户角色设置为非`admin`时，导致相关用户无法看见终端，部分场景下分析调试不方便

     ![非admin角色无法看见终端](/blog_img/devops/share-kubepshere-using-experience-for-current-company/no-admin-role-can-not-see-terminal.png "非admin角色无法看见终端") 

   * 公司网络环境多变且随着使用项目的增多，经常需要备份/迁移，目前的方式是手工备份`Jenkins`文件，但在`Jenkins`流水线构建参数无法体现在`Jenkins`流水线中，每次备份/迁移后都需要手工创建参数,此问题是我们目前的一大痛点，个人感觉`Zadig`做的不错(这部分感觉`Zadig`也是`KubeSphere`的有力竞争者)

     ![有多个构建参数的流水线](/blog_img/devops/share-kubepshere-using-experience-for-current-company/jenkins-pipeline-with-multiple-parameters.png "有多个构建参数的流水线") 

# 参考文章

* [KubeSphere使用心得](/post/devops/share-experiences-for-using-kubesphere/)
* [Kubesphere集成LDAP踩坑记录](/post/devops/setting-ldap-for-kubesphere/)
* [利用Nacos与KubeSphere创建多套开发与测试环境](/post/devops/using-nacos-and-kubesphere-to-create-multiple-environments/)
* [在Jenkins中根据配置从不同的仓库中Checkout代码](/post/devops/git-checkout-by-dynamic-repository-in-jenkins/)
* [由于k8s中的错误配置导致无法创建新节点](/post/k8s/cpu-resource-not-enough-due-to-invalid-k8s-config/)