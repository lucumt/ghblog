---
title: "给Yapi添加LDAP登录认证"
date: 2022-12-01T11:23:00+08:00
lastmod: 2023-04-14T11:23:00+08:00
draft: false
keywords: ["yapi","ldap","docker"]
description: "简要介绍如何给基于Docker创建的Yapi添加LDAP登录认证，以方面其使用"
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

`YApi` 是**高效**、**易用**、**功能强大**的 api 管理平台，旨在为开发、产品、测试人员提供更优雅的接口管理服务[^1]，由去哪儿网开发，本文基于[为 Yapi 定制 apline 版 Docker 镜像](https://www.jianshu.com/p/a97d2efb23c5)简要说明如何将基于`docker`安装的`Yapi`与`LDAP`集成。

<!--more-->

1. 利用`docker`创建[MongoDB](https://www.mongodb.com/)数据库

   ```bash
   docker run -d \
   --name mongo-yapi \
   -v $PWD/mongo-data:/data/db \
   -e MONGO_INITDB_ROOT_USERNAME=anoyi \
   -e MONGO_INITDB_ROOT_PASSWORD=anoyi.com \
   mongo
   ```

2. 在当前目录下创建一个`config.json`文件，内容如下

   ```json
   {
     "port": "3000",
     "adminAccount": "admin@anoyi.com",
     "timeout":120000,
     "db": {
       "servername": "mongo",
       "DATABASE": "yapi",
       "port": 27017,
       "user": "anoyi",
       "pass": "anoyi.com",
       "authSource": "admin"
     }
   }
   ```

3. 执行下述命令初始化 YAPI 数据库索引及管理员账号

   ```bash
   docker run -it --rm \
   --link mongo-yapi:mongo \
   --entrypoint npm \
   --workdir /yapi/vendors \
   -v $PWD/config.json:/yapi/config.json \
   anoyi/yapi \
   run install-server
   ```

   若执行成功会出现下述界面

   ![yapi初始化成功](/blog_img/ldap/add-ldap-support-for-yapi/docker-init-yapi-db-and-index.png "yapi初始化成功") 

4. 执行下述命令启动`yapi`

   ```bash
   docker run -d \
   --name yapi \
   --link mongo-yapi:mongo \
   --workdir /yapi/vendors \
   -p 3000:3000 \
   -v $PWD/config.json:/yapi/config.json \
   anoyi/yapi \
   server/app.js
   ```

   执行完毕后用`docker logs yapi`查看日志，若出现如下日志则表示启动成功

   ![yapi启动成功](/blog_img/ldap/add-ldap-support-for-yapi/docker-start-success-log.png "yapi启动成功") 

5. 根据上述提示在浏览器中打开`yapi`，其主页类似如下

   ![yapi主页](/blog_img/ldap/add-ldap-support-for-yapi/yapi-home-page.png "yapi主页") 

6. 点击`登录/注册` 之后会打开如下图所示的登录界面，可用管理员账号登录(用户名`admin@anoyi.com`、密码`ymfe.org`)，可发现此时其不支持`LDAP`登录

   ![yapi默认登录页](/blog_img/ldap/add-ldap-support-for-yapi/yapi-default-login-page.png "yapi默认登录页") 

7. 在`config.json`中添加`LDAP`相关的配置，之后输入`docker restart yapi && docker logs yapi`重启并观察日志

   ```json
   {
     "port": "3000",
     "adminAccount": "admin@anoyi.com",
     "timeout":120000,
     "db": {
       "servername": "mongo",
       "DATABASE": "yapi",
       "port": 27017,
       "user": "anoyi",
       "pass": "anoyi.com",
       "authSource": "admin"
     },
     "ldapLogin":{
        "enable":true,
        "server": "ldap://10.10.xx.xxx:389",
        "searchDn": "dc=xxx,dc=com",
        "searchStandard":"uid",
        "usernameKey":"uid"
      }
   }
   ```

8. 重新打开`yapi`，其登录页中会出现类似如下的`LDAP`选项，至此，操作完成！

   ![yapi支持ldap登录](/blog_img/ldap/add-ldap-support-for-yapi/yapi-ldap-login-page.png "yapi支持ldap登录") 

[^1]:http://yapi.smart-xwork.cn/