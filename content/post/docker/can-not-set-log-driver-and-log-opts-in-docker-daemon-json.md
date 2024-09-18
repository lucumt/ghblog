---
title: "由于daemon.json中的配置与其它启动项冲突导致docker服务无法启动"
date: 2022-08-04T16:56:18+08:00
lastmod: 2022-08-04T16:56:18+08:00
draft: false
keywords: ["docker"]
description: "记录由于/etc/docker/daemon.json与/etc/sysconfig/docker中的配置冲突导致docker服务无法启动的解决原因"
tags: ["docker"]
categories: ["容器化"]
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

由于公司的`docker`容器在运行一段时间后日志变得很大，通过`shell`脚本或者结合`docker stop`、`docker rm`和`docker run`来重新创建实例方式都觉得太麻烦，按照网络上的建议在`/etc/docker/daemon.json`中进行相关修改后却一直无法启动，同时错误信息一直提示`unable to configure the Docker daemon with file /etc/docker/daemon.json: the following directives are specified both as a flag and in the config…e: json-file)`，经过一番排查后终于找到原因，故记录下。

<!--more-->

## 问题复现

### 系统环境

* 操作系统，`uname -a`输出如下：

  ```tex
  Linux AEHPD 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
  ```

* `docker info`输出的主要信息如下:

  ```
  
  Containers: 0
   Running: 0
   Paused: 0
   Stopped: 0
  Images: 0
  Server Version: 1.13.1
  Storage Driver: overlay2
   Backing Filesystem: xfs
   Supports d_type: true
   Native Overlay Diff: true
  Logging Driver: journald
  Cgroup Driver: systemd
  Plugins:
   Volume: local
   Network: bridge host macvlan null overlay
  Swarm: inactive
  Runtimes: docker-runc runc
  Default Runtime: docker-runc
  ```

### 操作过程

1. 在`/etc/docker/daemon.json`的配置如下:

   ```json
   {
    "registry-mirrors":["https://docker.hirain.com"],
     "log-driver": "json-file",
     "log-opts": {
           "max-size": "50m",
           "max-file":"3"
     }
   }
   ```

2. 之后执行`systemctl daemon-reload && systemctl restart docker`会提示`docker`启动失败，根据输出提示利用`systemctl status docker.service`获取详细的输出信息如下:

   ![docker启动失败](/blog_img/docker/can-not-set-log-driver-and-log-opts-in-docker-daemon-json/docker-start-failed-when-add-json-config.png "docker启动失败") 

   从输出中可看见系统提示如下报错信息:

   >  unable to configure the Docker daemon with file /etc/docker/daemon.json: the following directives are specified both as a flag and in the config...e: json-file)

3. 基于前述步骤的提示信息，在`/etc/docker/daemon.json`移除`"log-driver": "json-file"`这行配置然后执行相关命令重启会发现 `docker`依旧启动失败！

   ![docker再次启动失败](/blog_img/docker/can-not-set-log-driver-and-log-opts-in-docker-daemon-json/docker-start-failed-when-add-json-config.png "docker再次启动失败") 

   从输出中可看见如下提示信息:

   > dockerd-current[14912]: Failed to set log opts: unknown log opt 'max-size' for journald log driver

4. 在`/etc/docker/daemon.json`将`log-opts`相关配置移除后，发现`docker`能启动成功！至此可以得出如下2条结论:

   * `log-driver`配置由于与其它指令冲突了，导致`docker`服务无法启动，后续去`docker`官网查询也确实有此说明[^1]
   * `log-opts`单独在`/etc/docker/daemon.json`也不生效

## 排查与解决

1. 在可正常启动`docker`服务的环境下利用`docker info --format '{{.LoggingDriver}}'命令发现其输出为`journald`，显然之前的`log-driver`配置与其冲突了，从而导致`docker`服务无法启动 

2. 前面的步骤中提示有`/etc/docker/seccomp.json `，查看其中的内容没有发现有价值的信息

3. 通过`systemctl status docker.service -l`输出如下，在其中发现关键信息`--log-driver=journald `，很明显是其它地方有对应设置

   ![docker服务详细日志输出](/blog_img/docker/can-not-set-log-driver-and-log-opts-in-docker-daemon-json/docker-service-detail-output.png "服务详细日志输出")  

4. 查找网络资料，利用`cat  /lib/systemd/system/docker.service`查看是由有其它配置文件，发现配置文件` /etc/sysconfig/docker`

   ![docker服务配置信息输出](/blog_img/docker/can-not-set-log-driver-and-log-opts-in-docker-daemon-json/docker-service-config-output.png "docker服务配置信息输出")  

5. 输出 `/etc/sysconfig/docker`结果如下，发现关键信息`--log-driver=journald `，它就是我们之前修改`/etc/docker/daemon.json`后一直无法启动的罪魁祸首！

   ![/etc/sysconfig/docker配置输出](/blog_img/docker/can-not-set-log-driver-and-log-opts-in-docker-daemon-json/docker-sysconfig-output.png "/etc/sysconfig/docker配置输出")

6. 接下来在`/etc/sysconfig/docker`中的相关位置将`--log-driver=journald `这条指令去掉，然后在`/etc/docker/daemon.json`中重新加上相关之后后`docker`容器可正常启动，问题解决！

[^1]: [Troubleshoot conflicts between the `daemon.json` and startup scripts](https://docs.docker.com/config/daemon/#troubleshoot-conflicts-between-the-daemonjson-and-startup-scripts)

