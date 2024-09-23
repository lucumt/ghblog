---
title: "给禅道软件集成LDAP实现单点登录"
date: 2022-12-20T10:04:00+08:00
lastmod: 2023-03-28T10:04:00+08:00
draft: true
keywords: ["ldap","禅道"]
description: "简要介绍基于Docker环境给禅道软件集成LDAP实现单点登录，主要涉及到对部分代码的定制化修改"
tags: ["ldap","docker"]
categories: ["工具使用","系统集成"]
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

---

[**禅道**](https://www.zentao.net/)是国产的开源项目管理软件,专注研发项目管理,内置需求管理、任务管理、bug管理、缺陷管理、用例管理、计划发布等功能,实现了软件的完整生命周期管理。本文基于[**开源版禅道集成LDAP实现统一身份认证**](https://www.jianshu.com/p/9981bdb608cb)一文简要说明如何将基于`docker`安装的`禅道`与`LDAP`集成实现快捷登录。

<!--more-->

1. 创建一个名为`zentao`的文件夹，在其下建立一个名为`docker-compose.yml`的文件，输入如下内容

   ```yaml
   version: '3'
   services:
     postgres:
       image: easysoft/zentao:latest
       restart: always
       privileged: true
       container_name: zentao
       ports:
         - 8282:80
         - 3336:3306
       volumes:
         - /www/zentaopms:/www/zentaopms
         - /www/mysqldata:/var/lib/mysql
       environment:
         TZ: Asia/Shanghai
         MYSQL_ROOT_PASSWORD: 123456
   ```

2. 输入`docker-compose up -d`启动容器，等待2-3分钟后利用`docker logs zentao`查看其日志，若日志中出现类似如下信息，则表示`禅道`初步安装成功

   ![docker中查看禅道日志](/blog_img/ldap/add-ldap-support-for-zentao/zentao-docker-log-with-error.png "docker中查看禅道日志") 

3. 输入`http://ip:8282`会出现`禅道`的安装界面，在此过程中会要求我们输入初始的管理员账号和密码，安装完毕后根据前述的账号和密码登录系统，然后仿照下图所示添加对应的`LDAP`插件，相关的插件可从

   ![禅道添加插件](/blog_img/ldap/add-ldap-support-for-zentao/zentao-install-plugin-from-local-disk.png "禅道添加插件") 

4. 测试

5. 测试

6. 测试

7. 测试

8. 测试

9. 测试