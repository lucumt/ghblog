---
title: "在KubeSphere的流水线中下载构建具有子模块的项目"
date: 2023-06-17T14:27:13+08:00
lastmod: 2023-06-17T14:27:13+08:00
draft: false
keywords: ["devops","jenkins","git","kubesphere","submodule","子模块"]
description: "记录如何在KubeSphere的流水线中下载构建具有子模块的项目"
tags: ["devops","jenkins","git","kubesphere"]
categories: ["工具使用","持续集成"]
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
centerImage: false
borderImage: true

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

在使用[**KubeSphere**](https://www.kubesphere.io/zh/)对公司项目进行持续集成时，有一个项目在`Gitlab`中采用了[**Submodules**](https://git-scm.com/book/en/v2/Git-Tools-Submodules)结构，导致原有的流水线不能直接使用，一番折腾后找到两种方案，其中一种比较优雅，简单记录下。

<!--more-->

## 没有子模块

在没有子模块时，自己是直接通过`KubeSphere`的图形界面操作，类似下图，只需根据实际情况填入必要的参数保存即可。

![利用KubeSphere图形界面配置Git下载](/blog_img/devops/checkout-project-with-submodule-in-kubesphere/kubesphere-git-checkout-config.png "利用KubeSphere图形界面配置Git下载") 

之后`KubeSphere`会自动生成对应的`Jenkins`代码：

```groovy
stage('拉取代码') {
    agent none
    steps {
        git(credentialsId: 'gitlab-account', url: 'http://gitlab.xxx.com/lucumt-group/system.git',
            branch: '$BRANCH_NAME', changelog: true, poll: false)
    }
}
```

可以看到生成的代码是通过调用`Jekins`中的`git`方法封装实现的，而不是原始的`git clone xxx`。

## 有子模块时

前述pipeline代码在没有submodules时能正常工作，但遇到有submodules的工程时，无法将其对应的子模块下载下来导致在后续的代码编译阶段出错。



最开始自己参考的是[**How do I git clone a repo, including its submodules**](https://stackoverflow.com/questions/3796927/how-do-i-git-clone-a-repo-including-its-submodules)中`--recurse-submodules`标识，想直接通过`git`命令直接下载，类似代码如下：

```groovy
stage('拉取代码') {
    agent none
    steps {
        sh '''git clone --recurse-submodules -j8 http://gitlab.xxx.com/lucumt-group/system.git	'''
    }
}
```

实际运行后很快发现问题：由于`Gitlab`需要授权访问导致请求被拒绝！可之前的为啥好使？因为之前的是通过`git`函数基于token的方式访问的，但直接在命令行中通过`git`命令访问的话，又没法使用token，而每次`git clone`时手工输入用户密码又很不方便，怎么办？



对`KubeSphere`和`Jenkins`进行深入研究，并结合实际使用流程后，确定了如下考量点：

1. 出于简化的考虑，`Gitlab`的用户名和密码不能每次都输入，故只能用原来的token方式
2. 单次`git clone`无法下载所有工程的话，可考虑分批多次下载，每次下载对应一个`Gitlab`工程地址
3. 去`Jenkins`官网查看是否有支持下载子模块的插件或函数

接下来基于2和3分别说明对应的实现。 

### 分批下载

既然没有submodules时直接下载没问题，那将子模块也当成一个独立的`Gitlab`工程(事实就是这样)再次复用之前的代码去下载，之后将其移动到对应的位置不就行了吗？有问题没，完全没有问题！相关代码实现如下：

```groovy
stage('拉取代码') {
    agent none
    steps {
        dir('system') {
            git(credentialsId: 'gitlab-token', url: 'http://gitlab.xxx.com/lucumt-group/system.git',
                branch: '$BRANCH_NAME', changelog: true, poll: false)
        }

        dir('module1') {
            git(credentialsId: 'gitlab-token', url: 'http://gitlab.xxx.com/lucumt-group/module_1.git', 
                branch: '$MODULE_BRANCH_1', changelog: true, poll: false)
        }

        dir('module2') {
            git(credentialsId: 'gitlab-token', url: 'http://gitlab.xxx.com/lucumt-group/module_2.git', 
                branch: '$MODULE_BRANCH_2', changelog: true, poll: false)
        }

        sh '''rm -rf system/module_1 system/module_2
			mv module1 system/
			mv module2 system/
			'''
    }
}
```

此种方式实现起来很简单，但存在以下缺陷：

1. 每个子工程都需要单独的配置与单独下载，工作量增加
2. 每个子工程的分支需要手工指定，无法进行原生关联，增加了复杂度
3. 子工程下载完毕后需要通过`shell`脚本将其移动到对应目录下，才能形成完整的项目结构

### 递归下载

基于`Jenkins`中的[**GitSCM**](https://www.jenkins.io/doc/pipeline/steps/params/gitscm/)，通过此种方式可以在下载主工程时将其附带的子模块功能一并下载下来，同时还能确保子模块的分支也是正确的，不需要显示指定，相对于前一种方式更简洁，最终代码如下：

```groovy
stage('拉取代码') {
    agent none
    steps {
        checkout([$class: 'GitSCM',
                  branches: [[name: '$BRANCH_NAME']],
                  doGenerateSubmoduleConfigurations: false,
                  extensions: [[$class: 'SubmoduleOption',
                                disableSubmodules: false,
                                parentCredentials: true,
                                recursiveSubmodules: true,
                                reference: '',
                                trackingSubmodules: false]], 
                  userRemoteConfigs: [[url: 'http://gitlab.xxx.com/lucumt-group/system.git',credentialsId:'gitlab-token']]])
    }
}
```

在上述代码中重点是通过`recursiveSubmodules`来递归下载子模块，通过`parentCredentials`来使用父模块下载时使用的token。



需要注意的是截止本文写作时`KubeSphere v3.3.1`不支持此种方式的图形化编辑，当采用图形化编辑时会出现雷系如下界面，没有正确的展示出实际配置的代码，此时若再次保存，会导致之前的配置丢失，故在此种方式下不能使用图形化方式对其进行二次修改。

![GitSCM在KubeSphere图形化编辑时的界面](/blog_img/devops/checkout-project-with-submodule-in-kubesphere/kubesphere-gitscm-checkout-config.png "GitSCM在KubeSphere图形化编辑时的界面") 