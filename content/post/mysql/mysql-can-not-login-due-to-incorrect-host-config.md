---
title: "MySQL中由于user表中错误的host配置导入无法登录数据库"
date: 2019-08-22T11:32:19+08:00
lastmod: 2019-08-22T11:32:19+08:00
draft: false
keywords: ["mysql","host","user"]
description: "MySQL中由于user表中错误的host配置导入无法登录数据库"
tags: ["mysql"]
categories: ["数据库"]
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

在给`MySQL`数据库进行配置时，对于`mysql`数据库表下的`user`表配置错误，导致无法通过命令登录进入`MySQL`数据库，查找网上的文档发现可通过**安全模式**进入，简单记录下。

<!--more-->

1. 公司某个项目采用的`MySQL`数据库不支持远程连接，于是个人登录后在`host`表进入如下修改:

   ![host中的错误配置](/blog_img/mysql/mysql-can-not-login-due-to-incorrect-host-config/mysql-host-wrong-config.png "host中的错误配置") 

2. 由于将`host`修改为了`%d`而正常的写法是`%`,从而导致在重启`MySQL`后无法登录

   ![mysql无法登录](/blog_img/mysql/mysql-can-not-login-due-to-incorrect-host-config/mysql-can-not-login.png "mysql无法登录") 

3. 由于`user`表主要用于控制用户能否远程登录以及其权限，当此表配置错误时会出现无法通过常规的`mysql -uroot -pxxx`的方式登录数据库修改其配置，此时只能通过**安全模式**进行登录。首先在终端输入`service mysql stop`关闭`MySQL`服务，接下来输入`mysqld_safe --skip-grant-tables`以安全模式开启`MySQL`服务

   ![mysql开启安全模式](/blog_img/mysql/mysql-can-not-login-due-to-incorrect-host-config/mysql-start-with-safe-mod.png "mysql开启安全模式") 

4. 接下来在另一个终端中输入`mysql -uroot -pxxx`即可正常登录，登录之后重新修改`mysql`数据库下的`user`表中的相关配置并重启`MySQL`服务即可。

   ![mysql通过安全模式登录](/blog_img/mysql/mysql-can-not-login-due-to-incorrect-host-config/mysql-login-with-safe-mod.png "mysql通过安全模式登录") 

参考文档:

1. https://stackoverflow.com/questions/48246503/user-table-not-having-root-user

