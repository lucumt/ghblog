---
title: "给Sonarqube软件集成LDAP实现单点登录"
date: 2022-12-04T11:29:12+08:00
lastmod: 2023-04-14T11:29:12+08:00
draft: false
keywords: ["ldap","docker","sonarqube"]
description: "简要介绍如何基于docker给Sonarqube软件集成LDAP实现单点登录"
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

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

[SonarQube](https://docs.sonarqube.org/latest/)是管理代码质量一个开放平台，可以快速的定位代码中潜在的或者明显的错误，本文简要说明如何将基于`docker`安装的`SonarQube`与`LDAP`集成实现快捷登录。

<!--more-->

1. 创建一个名为`sonarqube`的文件夹，在其下建立一个名为`docker-compose.yml`的文件，输入如下内容

   ```yaml
   version: '3'
   services:
     postgres:
       image: postgres:14.4
       restart: always
       privileged: true
       container_name: postgres
       ports:
         - 5432:5432
       volumes:
         - $PWD/postgres/postgres-data:/var/lib/postgresql/data
       environment:
         TZ: Asia/Shanghai
         POSTGRES_USER: sonar
         POSTGRES_PASSWORD: sonar123
         POSTGRES_DB: sonar
       networks:
         - sonar-network
     sonar:
       image: sonarqube:8.9-community
       restart: always
       container_name: sonar
       depends_on:
         - postgres
       volumes:
         - $PWD/sonarqube/extensions:/opt/sonarqube/extensions
         - $PWD/sonarqube/logs:/opt/sonarqube/logs
         - $PWD/sonarqube/data:/opt/sonarqube/data
         - $PWD/sonarqube/conf:/opt/sonarqube/conf
       ports:
         - 9990:9000
       environment:
         SONARQUBE_JDBC_USERNAME: sonar
         SONARQUBE_JDBC_PASSWORD: sonar123
         SONARQUBE_JDBC_URL: jdbc:postgresql://postgres:5432/sonar
       networks:
         - sonar-network
   networks:
     sonar-network:
       driver: bridge
   ```

2. 输入`docker-compose up -d`启动容器，等待2-3分钟后利用`docker logs sonar`查看其日志，若日志中出现类似如下信息，则表示`SonarQube`初步安装成功

   ![sonarqube启动成功](/blog_img/ldap/add-ldap-support-for-sonarqube/docker-start-sonarqube-success.png "sonarqube启动成功") 

3. 输入`http://ip:9990`可打开如下图所示的登录界面，默认的账号和密码均为`admin`，可用此账号登录

   ![sonarqube登录页面](/blog_img/ldap/add-ldap-support-for-sonarqube/sonarqube-login-page.png "sonarqube登录页面") 

4. 在`$PWD/sonarqube/conf`目录下建立一个名为`sonar.properties`的文件，写入类似如下内容

   ```properties
   #LDAP settings
   #admin
   sonar.security.realm=LDAP
   ldap.url=ldap://10.10.xxx.xxx:389
   ldap.bindDn=cn=xxx,dc=xxx,dc=com
   ldap.bindPassword=xxx
   
   ldap.user.baseDn=dc=xxx,dc=com
   ldap.user.request=(&(objectClass=inetOrgPerson)(uid={login}))
   ldap.user.realNameAttribute=displayName
   ldap.user.emailAttribute=mail
   ```

5. 输入`docker-compose restart`重启之后即可采用`LDAP`账户登录！