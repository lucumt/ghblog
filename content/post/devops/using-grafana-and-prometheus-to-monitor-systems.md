---
title: "利用Grafana和Prometheus对系统进行监控"
date: 2022-12-15T10:56:54+08:00
lastmod: 2022-12-15T10:56:54+08:00
draft: true
keywords: ["grafana","promethus","系统监控","docker"]
description: "简要介绍如何基于docker环境利用Grafana和Prometheus对系统进行监控"
tags: ["devops","grafana","docker"]
categories: ["工具使用","容器化"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
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
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

简要介绍如何在`docker`环境下利用[**Grafana**](https://grafana.com/)和[**Prometheus**](https://prometheus.io/)对系统进行监控。

<!--more-->

# 软件安装

在[**给Grafana软件集成LDAP实现单点登录**](/post/ldap/add-ldap-support-for-grafana/)一文中简要说明了如何基于`docker`安装`Grafana`，本节出于简化使用与维护的目的，将`Grafana`与`Prometheus`合并到一个`docker-compose.yml`文件中，相关的文件如下

`docker-compose.yml`文件



# 系统接入