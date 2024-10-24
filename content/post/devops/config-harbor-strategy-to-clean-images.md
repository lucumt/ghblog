---
title: "配置Harbor中镜像清理的策略实现定期自动清理冗余镜像"
date: 2024-10-24T10:09:39+08:00
lastmod: 2024-10-24T10:09:39+08:00
draft: false
keywords: ["harbor","镜像清理","定时任务",“自动清理”]
description: "基于个人实际使用经验，简要记录如何通过给Harbor项目配置镜像策略实现自动清理冗余的Docker镜像，达到节省磁盘空间的目的"
tags: ["harbor"]
categories: ["持续集成","容器化"]
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

基于个人实际使用经验，简要记录如何通过给`Harbor`项目配置镜像策略实现自动清理冗余的`Docker`镜像，达到节省磁盘空间的目的。

<!--more-->

个人当前使用的版本为`v2.10.3`，其并没有提供全局的镜像清理策略，只能基于单个项目进行。

1. 进入到某个具体的`Harbor`项目中，显示类似如下

   ![Harbor项目主页面](/blog_img/devops/config-harbor-strategy-to-clean-images/harbor-project-home-page.png "Harbor项目主页面") 

2. 点击`策略`菜单，进入对应的配置界面

   ![Harbor策略配置主页面](/blog_img/devops/config-harbor-strategy-to-clean-images/harbor-strategy-page.png "Harbor策略配置主页面") 

3. 在上图中点击`添加规则`按钮，出现类似如下界面

   ![Harbor添加策略主页面](/blog_img/devops/config-harbor-strategy-to-clean-images/harbor-add-strategy-page.png "Harbor添加策略主页面") 

   配置的关键在于其中的镜像清理策略

   ![Harbor镜像清理策略](/blog_img/devops/config-harbor-strategy-to-clean-images/harbor-image-clean-options.png "Harbor镜像清理策略") 

4. 个人项目中采用的策略为保留最近推送的若干个镜像，相关配置如下

   ![Harbor镜像清理策略示例](/blog_img/devops/config-harbor-strategy-to-clean-images/harbor-image-clean-options-example.png "Harbor镜像清理策略示例") 

5. 添加完毕的显示界面类似如下，创建好的规则默认不会自动执行，需要手工触发或采用定时任务触发，可点击图中的按钮手工触发

   ![Harbor添加镜像清理策略成功](/blog_img/devops/config-harbor-strategy-to-clean-images/harbor-add-strategy-result.png "Harbor添加镜像清理策略成功") 

6. 执行完毕的结果类似如下

   ![Harbor镜像清理策略执行结果](/blog_img/devops/config-harbor-strategy-to-clean-images/harbor-image-strategy-execution-result.png "Harbor镜像清理策略执行结果") 

7. 也可点击相关按钮配置定时执行策略

   ![Harbor镜像清理策略定时执行配置](/blog_img/devops/config-harbor-strategy-to-clean-images/harbor-image-strategy-scheduler-config.png "Harbor镜像清理定时执行配置") 

   相关的配置项如下

   ![Harbor镜像清理策略定时执行选项](/blog_img/devops/config-harbor-strategy-to-clean-images/harbor-image-strategy-scheduler-config-option.png "Harbor镜像清理定时执行选项") 

   配置完毕后即可定时执行，至此整个镜像定期自动清理策略配置完毕！