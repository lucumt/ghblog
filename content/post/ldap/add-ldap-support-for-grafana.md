---
title: "给Grafana软件集成LDAP实现单点登录"
date: 2022-12-11T10:39:47+08:00
lastmod: 2023-04-21T10:39:47+08:00
draft: false
keywords: ["ldap","grafana","docker"]
description: "简要介绍如何给基于Docker创建的Grafana软件集成LDAP实现单点登录"
tags: ["ldap","docker","grafana"]
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

[**Grafana**](https://grafana.com/)是一款用`Go`语言开发的开源数据可视化工具，可以做数据监控和数据统计，带有告警功能，本文简要说明如何将基于`docker`安装的`Grafana`与`LDAP`集成实现快捷登录。

<!--more-->

1. 创建一个名为grafana的文件夹，在其下建立一个名为`docker-compose.yml`的文件，输入如下内容

   ```yaml
   version: "3"
   services:
     grafana:
       image: grafana/grafana
       container_name: "grafana"
   	privileged: true
       ports:
         - "3000:3000"
       restart: always
       volumes:
         - "$PWD/grafana_data:/var/lib/grafana"
       environment:
         - GF_SECURITY_ADMIN_USER=admin
         - GF_SECURITY_ADMIN_PASSWORD=Pass@word
   ```

2. 输入下述命令创建相关的挂载目录

   ```bash
   mkdir grafana_data && chmod 777 grafana_data
   ```

3. 输入`docker-compose up -d`启动容器，等待2-3分钟后利用`docker logs grafana`查看其日志，若日志中出现类似如下信息，则表示`SonarQube`初步安装成功

    ![docker中启动grafana成功](/blog_img/ldap/add-ldap-support-for-grafana/grafana-docker-start-success-log.png "docker中启动grafana成功") 

4. 输入`http://ip:3000`可打开如下图所示的登录界面，采用前述设置的账号密码可正常登录

    ![grafana登录页](/blog_img/ldap/add-ldap-support-for-grafana/grafana-login-page.png "grafana登录页") 

5. 在`$PWD/grafana`目录下建立一个名为`ldap.toml`的文件，写入类似如下内容

   ```toml
   [[servers]]
   host = "10.xxx.xx.xx"
   port = 389
   use_ssl = false
   start_tls = false
   ssl_skip_verify = false
   bind_dn = "cn=xxx,dc=xxx,dc=com"
   bind_password = 'xxx'
   search_filter = "(uid=%s)"
   search_base_dns = ["dc=xxx,dc=com"]
   
   [servers.attributes]
   name = "givenName"
   surname = "displayName"
   username = "uid"
   #member_of = "cn"
   email =  "mail"
   
   [[servers.group_mappings]]
   group_dn = "grafana-admins"
   org_role = "Admin"
   
   [[servers.group_mappings]]
   group_dn = "grafana-editors"
   org_role = "Editor"
   
   [[servers.group_mappings]]
   group_dn = "*"
   org_role = "Viewer"
   ```

   同时将`docker-compose.yml`修改如下 

   ```yaml
   version: "3"
   services:
     grafana:
       image: grafana/grafana
       container_name: "grafana"
   	privileged: true
       ports:
         - "3000:3000"
       restart: always
       volumes:
         - "$PWD/grafana_data:/var/lib/grafana"
   	  - "$PWD/ldap.toml:/etc/grafana/ldap.toml"
       environment:
         - GF_SECURITY_ADMIN_USER=admin
         - GF_SECURITY_ADMIN_PASSWORD=Pass@word
   	  - GF_AUTH_LDAP_ENABLED=true
   ```

6. 输入`docker-compose restart`重启之后即可采用`LDAP`账户登录。

7. 若需要同时安装`Prometheus`，则可将`docker-compose.yml`修改为类似如下：

   ```yaml
   version: "3"
   services:
     prometheus:
       image: prom/prometheus:latest
       container_name: "prometheus"
       restart: always
       ports:
         - "9090:9090"
       volumes:
         - "./prometheus.yml:/etc/prometheus/prometheus.yml"
         - "./prometheus_data:/prometheus"
     grafana:
       image: grafana/grafana
       container_name: "grafana"
       ports:
         - "3000:3000"
       restart: always
       volumes:
         - "./grafana_data:/var/lib/grafana"
         - "./ldap.toml:/etc/grafana/ldap.toml"
       environment:
         - GF_SECURITY_ADMIN_USER=admin
         - GF_SECURITY_ADMIN_PASSWORD=Pass@word
         - GF_AUTH_LDAP_ENABLED=true
   ```

   