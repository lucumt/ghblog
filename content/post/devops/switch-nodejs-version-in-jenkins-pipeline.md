---
title: "在基于KubeSphere的Jenkins的流水线中切换nodejs版本"
date: 2024-01-11T10:17:42+08:00
lastmod: 2024-01-11T10:17:42+08:00
draft: false
keywords: ["kubesphere","jenkins","nodejs","pipeline","版本切换"]
description: "简要说明如何实现在基于KubeSphere的Jenkins的流水线中切换nodejs版本，以便实现更强的扩展性与通用性"
tags: ["kubesphere","jenkins"]
categories: ["工具使用","持续集成"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: true
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

近期有个项目组有个`Vue`项目需要在基于`KubeSphere`的`Jenkins`流水线中用高版本的`nodejs`进行编译，自己一开始尝试通过在流水线中直接升级`nodejs`版本，但并未成功，后来参考`KubeSphere`论坛上的相关说明才解决该问题，简单记录下。

<!--more-->

## 问题&尝试

项目中使用的`KubeSphere`版本为`v3.3.1`，基于`nodejs`环境对某个 `Vue3`项目进行编译打包时，`KubeSphere`提示构建过程出错。Google搜索一番过后，大部分都说要升级`nodejs`版本，自己当前的版本为`v10.16.3`，确实有些低，于是尝试用如下指令升级版本

```bash
rm -rf node_modules package-lock.json yarn.lock
npm cache clean --force

npm config set registry https://mirrors.xxx.com/repository/NPM/

npm install node@16.16.0
npm install
node -v
npm run build:qiankun
```

重新执行后依旧报错，同时`node -v`输出的结果依然是`v10.16.3`，版本不是预期的升级后版本。

为啥`npm install node@16.16.0`这条指令执行正常，而`node -v`输出的仍然为旧版本呢？

在构建之前添加如下两条指令进行分析

```bash
$PWD/node_modules/node/bin/node -v
node config list
```

输出结果如下

![npm调试输出](/blog_img/devops/switch-nodejs-version-in-jenkins-pipeline/npm-debug-output.png  "npm调试输出")

可以看出`npm install node@16.16.0`只是升级当前工程的`nodejs`版本，但是全局的版本还是旧的而编译时使用的是全局版本，导致问题出现。

将该指令变为`npm install node@16.16.0 -g`再次执行，结果提示权限不足

![npm全局安装报错](/blog_img/devops/switch-nodejs-version-in-jenkins-pipeline/npm-global-install-error.png  "npm全局安装报错")

至于为什么权限不足，原因也很简单，因为我们是在`Jenkins`流水线中执行该指令的，`Jenkins`每次执行都会创建一个临时目录，其出于安全考虑并没有赋予执行过程全部的权限。



通过`Jenkins`升级`nodejs`版本这条路走不通，那能否通过对相关工程源码进行修改，让其兼容低版本呢？内部讨论后出于下述考量，此方案也不可行：

1. 降低工程依赖的`nodejs` 版本会涉及到大量的功能逻辑，需要进行全面的测试，确保没有引入新问题
2. 后续其它工程模块可能也会使用高版本的`nodejs`，从出于扩展的角度考虑此问题早晚都要解决



软件开发领域理论上要求 **高版本要能兼容低版本**，能否通过统一升级`Jenkins`中所用的`nodejs`版本来解决此问题呢？虽然理论分析确实可行，不过此种方式只能升级到特定版本，无法同时兼容多个版本，同时所有的前端项目构建都会受到影响，灵活性不高。

既然遇到了这个问题，个人希望一次性解决，最好能在流水线中指定所需的版本，与其它的流水线完全隔离！

## 解决

由于自己采用的是`KubeSphere`来集成`Jenkins`，其他人是否遇到了类似问题？通过相关关键词搜索后发现一篇文章[**kubesphere3.1.1，devops工程如何升级node.js**](https://ask.kubesphere.io/forum/d/6859-kubesphere311devopsnodejs)，其遇到的问题和我自己的很类似。

根据该问题中的回复，需要对`Jenkins`流水线文件头部的配置进行修改

原始配置如下

```groovy
agent {
    node {
        label 'nodejs'
    }
}
```

修改为

```groovy
agent {
    kubernetes {
        inheritFrom 'nodejs base'
        containerTemplate {
            name 'nodejs'
            image 'node:16.16.0'
        }
    }
}
```

同时将前端构建指令修改如下

```bash
rm -rf node_modules package-lock.json yarn.lock
npm cache clean --force

npm config set registry https://mirrors.xxx.com/repository/NPM/
npm install
node -v
npm run build:qiankun
```

再次执行相关构建，输出类似如下，可以看出`nodejs`版本已经修改为我们希望的版本，且能正常编译通过

![node版本升级](/blog_img/devops/switch-nodejs-version-in-jenkins-pipeline/node-version-upgraded.png  "node版本升级")

基于上述方式修改后，初次执行时可能会出现类似如下错误，出错的原因是相关镜像还没下载下来，多执行几次即可。

![kubernetes执行报错](/blog_img/devops/switch-nodejs-version-in-jenkins-pipeline/kubernetes-agent-not-working.png  "kubernetes执行报错")



当流水线全部执行完毕时，发现无法识别`docker`指令，导致无法进行编译构建。

![docker指令无法识别](/blog_img/devops/switch-nodejs-version-in-jenkins-pipeline/docker-command-not-found.png  "docker指令无法识别")

继续在该问题下寻找答案，发现`KubeSphere`官方的技术支持人员有给出答案

![kubesphere技术支持人员回复](/blog_img/devops/switch-nodejs-version-in-jenkins-pipeline/kubesphere-support-reply.png  "kubesphere技术支持人员回复")

将`docker`构建阶段的容器从`nodejs`修改为`base`即可，重新执行流水线可正常执行[^1]，至此问题解决完毕。

```bash
stage('镜像构建') {
    agent none
        steps {
            container('base') {
            sh '''echo $NODE_PORT
            cat cicd/Dockerfile
            docker build -f cicd/Dockerfile  --build-arg  PRODUCT_PHASE=$PRODUCT_PHASE -t orienlink-evaluator-web:$BUILD_TAG .'''
            }
        }
    }
}
```

## 后记

虽然此文以`nodejs`举例，但对于其它的语言也可适用，假设使用的是`Golang`，对应的`Jenkins`流水线文件头部可修改如下

```groovy
agent {
    kubernetes {
        inheritFrom 'go base'
        containerTemplate {
            name 'go'
            image 'go:1.22.2'
        }
    }
}
```

通过上述方式可实现基于流水线级别的版本变更与隔离，使用起来更加方便。

[^1]: 完整的`Jenkins`流水线参见[lucumt-nodejs-version-change.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/lucumt-nodejs-version-change.groovy)