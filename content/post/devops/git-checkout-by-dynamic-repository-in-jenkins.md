---
title: "在Jenkins中根据配置从不同的仓库中Checkout代码"
date: 2023-03-10T22:48:50+08:00
lastmod: 2023-03-10T22:48:50+08:00
draft: false
keywords: ["kubesphere","jenkins","git","gitlab"]
description: "在Jenkins中根据配置从不同的仓库中Checkout代码减少流水线数目，简化开发与维护成本"
tags: ["kubesphere","jenkins","git"]
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

mermaidDiagrams: 
  enable: true
  options: "{
     'theme':'forest',
     'themeVariables': {
        'width':'60%',
        'lineColor': '#0a908e',
        'fontSize':'14px',
        'titleColor':'#eb071e',
        'tertiaryTextColor':'#eb071e'
     }
  }"
---

<style>
div.mermaid > svg { max-width: 60% !important; }
</style>

简要介绍如何在[KubeSphere](https://kubesphere.io/zh/)中使用[Jenkins](https://www.jenkins.io/)基于用户选择的不同[GitLab](https://gitlab.com)工程模块利用[Git](https://git-scm.com/)进行动态的代码下载和编译构建，**将多条流水线缩减为一条，减少`Jenkins`流水线的开发与维护成本**。

<!--more-->

## 背景

在[利用Nacos与KubeSphere创建多套开发与测试环境](/post/devops/using-nacos-and-kubesphere-to-create-multiple-environments/)一文中介绍了部门基于`KubeSphere`和`Nacos`来动态的创建多套开发与测试环境，此种方式虽然在公司内部使用起来很灵活，但当需要交付给客户时会存在如下问题导致`KubeSphere`不适合部署给客户使用:

1. 客户使用环境的网络无法接入公司网络，无法下载代码
2. 客户使用环境不一定有`Kubernetes`环境，无法部署pod节点

由于`docker`的安装相对`Kubernetes`而言会简单很多，故决定采用`docker`容器的形式在客户现场部署运行软件，相关流程如下：
![适应客户使用环境的软件部署流程](/blog_img/devops/git-checkout-by-dynamic-repository-in-jenkins/software-deploy-process-in-customer-environment.png "适应客户使用环境的软件部署流程") 

实际使用时在公司内部提前打包好对应的`docker`镜像并导出，然后在客户环境导入对应的`docker`镜像并运行即可，通过此种方式可有效的避免对于公司内部特定环境和配置的依赖。



为实现上述目的，可通过[docker save](https://docs.docker.com/engine/reference/commandline/save/)先导出镜像，然后使用`Jenkins`的[archiveArtifacts](https://www.jenkins.io/doc/pipeline/steps/core/#archiveartifacts-archive-the-artifacts)功能导出镜像文件即可，这样看来问题似乎很容易解决，直接修改原有的`Jenkins`流水线即可。由于不同项目模块在使用上会有细微的差异，同时处于简化开发和维护的考虑，目前部门的使用方式是**每个工程对应一条`Jenkins`流水线**，这样导致每条流水线都需要修改，工作量且不利于后续的扩展！

![每个工程模块均对应一条流水线](/blog_img/devops/git-checkout-by-dynamic-repository-in-jenkins/each-project-has-a-jenkins-pipeline.png "每个工程模块均对应一条流水线") 

基于上述原因，最终的实现方案如下

{{% admonition info "实现方案" %}} 

**部门内部研发测试使用的构建流水线与要交付给客户的产品构建流水线分开创建与使用，且交付构建的流水线数目要尽可能少，便于后续维护与定制化扩展**

 {{% /admonition %}}

下图展示了更详细的使用流程，在该图中引出了我们的问题

{{% admonition question "问题" %}} 

**如何在Jenkins中只通过一条流水线根据用户选择的工程模块来动态的下载代码?**

 {{% /admonition %}}

```mermaid
flowchart TD
    subgraph   
        A1(开始构建) -->A2{选择对应参数}
        A2 -->|用于获取端口等配置| A3[打包环境]
        A2 --> A4[选择工程]:::checkstyle
        A2 -->|对应工程Gitlab分支| A5[选择分支]:::checkstyle
        A4 --> A6[[Checkout代码]]:::checkstyle
        A5 --> A6
        A6 --> |更新yaml等配置文件| A7[更新配置]
        A3 --> A7
        A7 --> A8[[代码编译]]
        A8 --> A9[(导出jar文件)]
        A8 --> A10[镜像构建]
        A10 --> A11[(导出镜像)]
        A9 --> A12(((构建结束)))
        A11 --> A12
    end
    subgraph  
        B1(导入镜像) --> B2[文件校验] --> B3[[容器部署]]
    end
    subgraph  
        C1(导入jar) --> C2[文件校验] --> C3[[运行jar]]
    end
    A11 -->|docker方式部署| B1
    A9 --> |直接运行jar文件| C1
classDef checkstyle fill:#6bacf2,color:#ffffff,font-weight:bold
```

## 解决思路

由于最初是每个工程项目都有一个单独的`Jenkins`流水线，故在通过`Git`下载代码时采用的是类似下图的直接写死`Gitlab`工程仓库地址的方式，很明显此种方式不满足要求。

```groovy
stage('拉取代码') {
    agent none
    steps {
        // git仓库地址直接以字符串方式写死
        git(credentialsId: 'gitlab-account', url: 'http://gitlab.xxx.com/lucumt-group/system.git', branch: '$BRANCH_NAME', changelog: true, poll: false)
    }
}
```

在网上搜索后发现一篇文章[dynamically-selecting-git-repo-in-jenkins-job](https://stackoverflow.com/questions/56421553/dynamically-selecting-git-repo-in-jenkins-job),其采用`parameters`实现，将`Gitlab`仓库的地址存储在一个变量中，然后`Git`下载时实际计算变量值并进行下载。此方案看起来符合要求，进一步查找后发现`parameters`本身无法直接修改[^1]，不满足使用要求。

由于先前对代码分支名称的获取可通过`$BRANCH_NAME`来实现，自己尝试采用类似的方式来验证，最终找到了符合要求的实现方案

![采用不同方式尝试在jenkins中实现动态下载](/blog_img/devops/git-checkout-by-dynamic-repository-in-jenkins/jenkins-pipeline-dynamic-checkout-code-testing-result.png "采用不同方式尝试在jenkins中实现动态下载") 

下图展示了完整的使用链路，可发现其实现复杂度不是很高

![jenkins动态下载代码图解](/blog_img/devops/git-checkout-by-dynamic-repository-in-jenkins/jenkins-dynamic-select-git-repository-explanation.png "jenkins动态下载代码图解") 

## 展示效果

运行结果如下：

![采用jenkins动态下载代码并构建的示例](/blog_img/devops/git-checkout-by-dynamic-repository-in-jenkins/jenkins-dynamic-checkout-code-and-build-example.png "采用jenkins动态下载代码并构建的示例") 

相关流水线参考如下：

| 流水线                                                       | 作用                                       | 备注                        |
| ------------------------------------------------------------ | ------------------------------------------ | --------------------------- |
| **[lucumt-server-image-build.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/lucumt-server-image-build.groovy)** | 服务器端打包流水线，产出物为**docker镜像** |                             |
| **[lucumt-front-image-build.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/lucumt-front-image-build.groovy)** | 前端打包流水线，产出物为**docker镜像**     |                             |
| **[lucumt-server-jar-build.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/lucumt-server-jar-build.groovy)** | 服务器端打包流水线，产出物为**jar文件**    | 可直接用`Java`运行该jar文件 |
| **[lucumt-front-dist-build.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/lucumt-front-dist-build.groovy)** | 服务器端打包流水线，产出物为**zip文件**    | `nodejs`编译后的文件压缩    |


[^1]:https://stackoverflow.com/questions/61789938/change-jenkins-param-variable-value



