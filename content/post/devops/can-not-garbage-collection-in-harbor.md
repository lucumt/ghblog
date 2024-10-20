---
title: "Harbor清理垃圾时一直提示10010 object is not found导致清理失败"
date: 2024-10-16T15:03:19+08:00
lastmod: 2024-10-16T15:03:19+08:00
draft: true
keywords: ["Harbor","Garbage Collection","清理垃圾","失败","object is not found"]
description: "基于自己实际的使用经验，记录通过Harbor清理垃圾时遇到10010错误提示object is not found导致垃圾清理失败时的解决方案"
tags: ["devops","harbor"]
categories: ["持续集成","容器化"]
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

记录`Harbor`清理镜像垃圾时遇到的错误`{"errors":[{"code":"NOT_FOUND","message":"{\"code\":10010,\"message\":\"object is not found\",\"details\":\"2cc49b89751876fa18acb008\"}"}]}`导致回收失败，不能正常清理，最终占用越来越多的磁盘空间的解决过程。

<!--more-->

## 问题

`Harbor`整体占用了500多G的磁盘空间

![Harbor占用了大量的存储空间](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor_disk_usage_large.png "Harbor占用了大量的存储空间")  

对相关项目中的多余镜像手动删除后，项目下的磁盘占用空间恢复正常

![Harbor单个项目占用的磁盘空间](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor_project_disk_usage.png "Harbor单个项目占用的磁盘空间")  

但回到项目整体界面上去看时，其占用的总空间还是和之前的一样都为500多G。

根据之前的经验虽然镜像被删除了，但还存在大量的垃圾导致占用了巨量的存储空间，此时需要对其进行垃圾清理，在`系统管理`->`清理任务`中点击`立即清理垃圾`后经过几个小时后去查看结果时，类似如下，一直提示垃圾清理失败！

![Harbor清理垃圾失败](/blog_img/devops/can-not-garbage-collection-in-harbor/harbar_garbage_collection_failed.png "Harbor清理垃圾失败")  

点击某条具体的执行记录，查看其日志，显示如下，并没有啥有价值的信息

![Harbor清理垃圾错误日志](/blog_img/devops/can-not-garbage-collection-in-harbor/harbar_garbage_collection_error_log.png "Harbor清理垃圾错误日志")  

## 解决

根据前述报错信息去Google查找，发现这个问题很普遍

![相关错误Google搜索结果](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor-garbage-collection-failed-search-result.png "相关错误Google搜索结果")  

基于[这条回复](https://github.com/goharbor/harbor/issues/13440#issuecomment-721318580)、[这条回复](https://github.com/goharbor/harbor/issues/13732#issuecomment-743085175)以及[这篇文章](https://blog.51cto.com/foxhound/2482127)上说的是需要将`registry`数据库下的`admin_job`表的`deleted`字段从false修改为true，当前使用的`Harbor`版本为`v2.10.3`基于`docker`部署，进入名为`harbor-db`的容器后发现由于自己的`Harbor`版本相对于别人反馈的版本更新吗，该表并不存在！

![admin job表不存在](/blog_img/devops/can-not-garbage-collection-in-harbor/admin_job_table_not_exists.png "admin job表不存在") 

对`blob`表查询其总数，结果如下

![blob总数查询](/blog_img/devops/can-not-garbage-collection-in-harbor/postgres-blob-data-count.png "blob总数查询") 