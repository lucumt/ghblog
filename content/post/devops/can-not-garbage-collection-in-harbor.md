---
title: "Harbor清理垃圾时一直提示10010 object is not found导致清理失败"
date: 2024-10-23T15:21:19+08:00
lastmod: 2024-10-23T15:21:19+08:00
draft: false
keywords: ["Harbor","Garbage Collection","清理垃圾","失败","object is not found"]
description: "基于自己实际的使用经验，记录通过Harbor清理垃圾时遇到10010错误提示object is not found导致垃圾清理失败时的解决方案"
tags: ["devops","harbor"]
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

记录`Harbor`清理镜像垃圾时遇到的错误`{"errors":[{"code":"NOT_FOUND","message":"{\"code\":10010,\"message\":\"object is not found\",\"details\":\"2cc49b89751876fa18acb008\"}"}]}`导致回收失败，不能正常清理，最终占用越来越多的磁盘空间的解决过程。

<!--more-->

## 问题

通过监控工具发现服务器的磁盘利用率接近80%，导致部分`Kubernetes`节点不能正常工作，需要清理出一部分磁盘空间。进一步分析后发现`Harbor`整体占用了500多G的磁盘空间。

![Harbor占用了大量的存储空间](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor_disk_usage_large.png "Harbor占用了大量的存储空间")  

对`Harbor`中相关项目下的多余镜像手动删除后，项目下的磁盘占用空间恢复正常。

![Harbor单个项目占用的磁盘空间](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor_project_disk_usage.png "Harbor单个项目占用的磁盘空间")  

但回到项目整体界面上去看时，其占用的总空间还是和之前的一样都为500多G。

根据之前的经验虽然镜像被删除了，但还存在大量的垃圾导致占用了巨量的存储空间，此时需要对其进行垃圾清理，在`系统管理`->`清理任务`中点击`立即清理垃圾`后经过几个小时后去查看结果时，类似如下，一直提示垃圾清理失败！

![Harbor清理垃圾失败](/blog_img/devops/can-not-garbage-collection-in-harbor/harbar_garbage_collection_failed.png "Harbor清理垃圾失败")  

一开始自己以为是要清理的镜像垃圾太多导致需要的时间也更多，但自己等了两三天还是清理失败，这肯定不正常！

点击某条具体的执行记录，查看其日志，显示如下，并没有啥有价值的信息

![Harbor清理垃圾错误日志](/blog_img/devops/can-not-garbage-collection-in-harbor/harbar_garbage_collection_error_log.png "Harbor清理垃圾错误日志")  

## 解决

根据前述报错信息去Google查找，发现在`GitHub`上这个问题很普遍

![相关错误Google搜索结果](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor-garbage-collection-failed-search-result.png "相关错误Google搜索结果")  

基于[这条回复](https://github.com/goharbor/harbor/issues/13440#issuecomment-721318580)、[这条回复](https://github.com/goharbor/harbor/issues/13732#issuecomment-743085175)以及[这篇文章](https://blog.51cto.com/foxhound/2482127)上说的是需要将`registry`数据库下的`admin_job`表的`deleted`字段从false修改为true，当前使用的`Harbor`版本为`v2.10.3`基于`docker`部署，进入名为`harbor-db`的容器后发现由于自己的`Harbor`版本相对于别人反馈的版本更新，该表并不存在！

![admin job表不存在](/blog_img/devops/can-not-garbage-collection-in-harbor/admin_job_table_not_exists.png "admin job表不存在") 

对`blob`表查询其总数，结果如下，虽然总数很多，但并没有提供啥有用的信息

![blob总数查询](/blog_img/devops/can-not-garbage-collection-in-harbor/postgres-blob-data-count.png "blob总数查询") 

继续查看对应的搜索结果，在[这条回复](https://github.com/goharbor/harbor/issues/20821#issuecomment-2283371901)中找到一条相关度很高的说明

> Was this job executed 24 hours ago, the job service reaper all the execution and task exceed 24 hour and result in the not found error.
>
> Please update the configuration and restart Harbor. use the file logger for job service and change the max_update_hours to 168

该段回复的主要意思是**清理垃圾的时间太长超过了24小时，任务中心将该任务进行了强行终止与回收，进而导致非正常的查询失败错误。**

而我之前也是等待了两三天也没清理成功，于是按照其说明修改`$PWD/common/config/jobservice/config.yml`，将对应时间从**24**修改为**168**,之后重启`Harbor`服务。

```yaml
jobLoggers:
    - file
 ...

reaper:
   max_update_hours: 168
   max_dangling_hours: 168
```

基于`Docker`重启`Harbor`服务之后，我又等待了两三天，结果依旧和以前一样！

随着日常开发任务的迭代，`Harbor`镜像仓库占用的磁盘空间也越来越大，该问题必须解决，没办法，我只能硬着头皮继续翻看`GitHub`中关于这方面问题的相关检索结果。

最终在[这条回复](https://github.com/goharbor/harbor/issues/20821#issuecomment-2275160905)中找到`Harbor`开发者的如下说明

> Please go to the Job Service Dashboard and select the workers and you can find the job in the worker, and there is a log link, you can check when it is running

切换到自己的`Harbor`对应界面后发现任务果然被挂起

![Harbor任务被挂起](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor-task-suspended.png "Harbor任务被挂起") 

选中`GARBAGE_COLLECTION`类型的任务，将其重新启动，然后切换到`工作者(Worker)`tab页下，可发现其下有任务正在执行。

![Harbor任务执行者查看](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor-work-check.png "Harbor任务执行者查看") 

点击对应的链接，可查看该任务执行者的详细信息，类似如下，从中可发现此时已经开始正常清理镜像垃圾！

![Harbor任务清理详细日志](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor_garbage_collection_detail_logs.png "Harbor任务清理详细日志") 

在`Harbor`界面查看相关任务执行结果，发现其状态均变为`SUCCESS`

![Harbor任务执行成功](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor-task-running-success.png "Harbor任务执行成功") 

点击其中某一条任务查看详细日志，结果如下，此时可发现令人讨厌的`not found`错误已经消失!

![Harbor具体任务执行日志](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor-specific-task-log.png "Harbor具体任务执行日志") 

在`Harbor`主界面查看最关键的磁盘空间占用，已经减少到不到200G，符合预期，问题解决！

![Harbor镜像占用的磁盘空间显著减少](/blog_img/devops/can-not-garbage-collection-in-harbor/harbor_disk_usage_reduced.png "镜像占用的磁盘空间显著减少") 