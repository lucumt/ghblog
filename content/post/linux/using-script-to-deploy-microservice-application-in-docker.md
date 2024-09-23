---
title: "利用shell脚本实现将微服务程序以docker容器方式自动部署"
date: 2023-03-16T11:22:47+08:00
lastmod: 2023-03-16T11:22:47+08:00
draft: false
keywords: ["linux","docker","shell","微服务"]
description: "利用shell脚本实现将微服务程序以docker容器方式自动部署，主要适用于ToB软件在客户现场部署的场景"
tags: ["linux","docker","shell"]
categories: ["脚本操作","持续集成"]
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

mermaidDiagrams: 
  enable: true''
  options: "
     {'theme':'forest'}
  "
---

基于[在Jenkins中根据配置从不同的仓库中Checkout代码](/post/devops/git-checkout-by-dynamic-repository-in-jenkins/)一文，说明如何利用`shell`脚本在`Linux`系统中实现自动化的部署与升级。

<!--more-->

## 流程图

整体流程如下图所示：

1. 首先让用户选择要升级的类别和模块，并对用户选择进行校验，若不合法则升级过程终止
2. 根据用户选择去相关目录下查看对应的文件是否存在，若不存在则升级过程终止
3. 若文件存在则对其进行解压，并读取其中的配置文件信息[^1]
4. 检查对应的容器，将其停止并删除
5. 导入相关镜像
6. 拼接`docker`语句并执行，展示执行结果

<style>
div.mermaid > svg { max-width: 70% !important; }
</style>

```mermaid
flowchart TD
    START((开始部署)):::start
    FAIL((部署失败)):::failed
    SUCCESS((部署成功)):::success
    subgraph deploy_select [输入选择]
        A1[选择类别]:::input --> A2{类别是否有效}
        A2 -->|类别有效| A3[选择模块]:::input
        A3 --> A4{模块是否有效}
        A4 -->|模块有效| A5[(记录用户选择)]
    end
    subgraph parse_file [解析文件]
       B1[查找文件] --> B2{文件是否存在}
       B2 -->|文件存在| B3[解压文件]
       B3 --> B4[读取ini配置文件]
       B4 --> B5[/输出用户选择与配置文件信息/]
    end
    subgraph deploy_process [开始部署]
       C1{容器是否运行} -->|容器运行|C2[停止容器]
       C1 -->|容器未运行| C3[删除容器]
       C2 --> C3
       C3 --> C4[导入镜像]:::input
       C4 --> C5[拼接docker字符串]
       C5 --> C6[打印要执行的docker语句]
       C6 --> C7[执行docker语句]
       C7 --> C8{docker容器是否执行成功}
       C8 -->|docker执行成功|C9
    end
    START --> A1
    A2 -->|类别无效| FAIL
    A4 -->|模块无效| FAIL
    A5 --> B1
    B2 -->|文件不存在|FAIL
    B5 --> C5
    C8 -->|docker执行失败|FAIL
    C9 --> SUCCESS
    classDef start fill:#374bad,color:#ffffff,font-weight:bold,stroke-width:0
    classDef failed fill:#f5425d,color:#ffffff,font-weight:bold,stroke-width:0
    classDef input fill:#ad5537,color:#ffffff,stroke-width:0
    classDef success fill:#37ad61,color:#ffffff,font-weight:bold,stroke-width:0
```

## 使用说明

此部分操作主要基于前述的流程以`shell`脚本的形式实现，相关源码参见[**deploy.sh**](https://github.com/lucumt/myrepository/blob/master/linux/docker_image_deploy.sh)，具体操作流程如下：

* 将`deploy.sh`拷贝到`Linux`服务器的某个目录下，然后在该目录下建立一个名为**`target`**的目录，将导出的镜像文件放到`target`目录下

  ![服务中要升级的镜像与脚本目录结构](/blog_img/linux/using-script-to-deploy-microservice-application-in-docker/deploy-shell-and-images-in-linux-folder.png "服务中要升级的镜像与脚本目录结构")

* 执行`./deploy`命令并按下`Enter`键，终端会出现如下输出，让我们选择要升级的系统为前端还是后端

  ![选择要升级的类型](/blog_img/linux/using-script-to-deploy-microservice-application-in-docker/deploy-shell-ask-deploy-type.png "选择要升级的类型")

* 根据提示选择对应的类别，若升级类型为前端则输入1，若升级类型为后端则输入2，之后按下Enter键则系统会进一步提示选择对应的模块

  ![选择要升级的模块](/blog_img/linux/using-script-to-deploy-microservice-application-in-docker/deploy-shell-ask-deploy-module.png "选择要升级的模块")

  若输入的类型不合法，则升级脚本会给出对应的提示，同时整个升级过程立即终止

  ![错误的输入参数导致升级中断](/blog_img/linux/using-script-to-deploy-microservice-application-in-docker/deploy-shell-input-invalid-deploy-type.png "错误的输入参数导致升级中断")

* 若要部署的模块在`target`目录下不存在或选择的模块不是选择列表的模块，升级脚本同样会给出提示，同时整个升级过程立即终止

  ![找不到要升级的文件](/blog_img/linux/using-script-to-deploy-microservice-application-in-docker/deploy-shell-can-not-find-module-file.png "找不到要升级的文件")

  若部署的模块在`target`目录下存在多份，则会根据文件名中附带的时间戳进行排序，寻找最近的一份文件进行升级

  ```bash
  # 校验升级文件是否存在
  file=$(ls target|sort -r|grep ${module}_20|head -1) # 此条指令根据文件名按时间倒排，取第一个文件来升级
  if [[ -z "$file" ]]
  then
  printf "\033[31mtarget目录下没有对应的文件，升级操作终止，请重新执行./deploy.sh\033[0m\n"
  exit 0
  fi
  ```

* 若选择的模块合法，且升级过程一切顺利，Linux终端会输出类似如下信息，提示整个升级过程完成!

  ![升级过程顺利执行完毕](/blog_img/linux/using-script-to-deploy-microservice-application-in-docker/deploy-shell-execute-success.png "升级过程顺利执行完毕")

* 若升级过程中遇到错误，系统同样会给出提示，可根据错误信息进行初步排查

  ![升级过程执行失败](/blog_img/linux/using-script-to-deploy-microservice-application-in-docker/deploy-shell-execute-failed.png "升级过程执行失败")

## 待优化点

* 升级过程缺少记录，无法查看在特定时间范围通过该脚本执行升级的次数，可通过写入文件实现简单的数据库记录
* 缺少回退功能，在最后启动`docker`容器失败是，应能将之前关闭的服务启动，确保能持续对外提供服务
* 对于集群方式部署的`docker`容器缺少兼容性

[^1]:此信息在部署过程中会输出，用于确认版本和配是否正确