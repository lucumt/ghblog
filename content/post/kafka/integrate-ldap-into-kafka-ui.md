---
title: "在基于Docker搭建的Kafka UI中整合LDAP"
date: 2023-11-20T15:37:31+08:00
lastmod: 2023-11-20T15:37:31+08:00
draft: false
keywords: ["docker","kafka","ldap"]
description: "在基于Docker搭建的Kafka UI中整合LDAP，供学习参考使用"
tags: ["docker","kafka","ldap"]
categories: ["容器化","工具使用","消息队列"]
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

简要记录如何基于`docker`搭建`Kafka`服务器以及添加集成了`LDAP`的`kafka-ui`实现图形化界面的授权访问。

<!--more-->

## kafka安装

基于`docker-compose`的方式安装，脚本如下

```yaml
version: "3"
services:
  zookeeper:
    restart: always
    image: docker.io/bitnami/zookeeper:3.8
    #network_mode: "bridge"
    container_name: zookeeper_test
    ports:
      - "2181:2181"
    volumes:
      - $PWD/zk_data:/bitnami/zookeeper #持久化数据
    environment:
      - TZ=Asia/Shanghai
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    restart: always
    image: docker.io/bitnami/kafka:3.4.1
    #network_mode: "bridge"
    container_name: kafka_test
    ports:
      - "9004:9004"
    volumes:
      - $PWD/kafka_data:/bitnami/kafka #持久化数据
    environment: 
      - TZ=Asia/Shanghai - KAFKA_BROKER_ID=1 
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9004
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://10.10.2.98:9004 #替换成你自己的IP
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181 
      - ALLOW_PLAINTEXT_LISTENER=yes 
    depends_on: 
      - zookeeper
```

## kafka-ui的安装

参考[**kafka-ui**](https://github.com/provectus/kafka-ui)的说明，基于`docker-compose`的方式安装，脚本如下

```yaml
version: "3"
services:
  kafka-ui:
    restart: always
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    restart: always
    ports:
      - 9001:8080
    environment:
      - KAFKA_CLUSTERS_0_NAME=kafka-test
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=10.10.2.98:9004
      - KAFKA_CLUSTERS_0_ZOOKEEPER=10.10.2.98:2181
```

之后可通过`http://SERVER_IP:9001`访问，界面类似如下，

![Kafka UI没有添加认证](/blog_img/kafka/integrate-ldap-into-kafka-ui/kafka-ui-without-auth.png "Kafka UI没有添加认证") 

此时同网络下的任何人都能访问，也能通过UI界面对其进行相关修改操作，缺乏权限控制。

## 添加登录

### 普通登录

普通登录方式的配置脚本如下，此时其账户信息以硬编码的形式存在

```yaml
version: "3"
services:
  kafka-ui:
    restart: always
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    restart: always
    ports:
      - 9001:8080
    environment:
      - KAFKA_CLUSTERS_0_NAME=kafka-test
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=10.10.2.98:9004
      - KAFKA_CLUSTERS_0_ZOOKEEPER=10.10.2.98:2181
      - AUTH_TYPE="LOGIN_FORM"
      - SPRING_SECURITY_USER_NAME=admin
      - SPRING_SECURITY_USER_sPASSWORD=123456
```

对应的登录界面如下:

![Kafka UI登录认证](/blog_img/kafka/integrate-ldap-into-kafka-ui/kafka-ui-login-panel.png "Kafka UI登录认证") 

### LDAP登录

`LDAP`登录方式的配置脚本如下，其登录界面与前述一样

```yaml
version: "3"
services:
  kafka-ui:
    restart: always
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    restart: always
    ports:
      - 9001:8080
    environment:
      - KAFKA_CLUSTERS_0_NAME=kafka-test
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=10.10.2.98:9004
      - KAFKA_CLUSTERS_0_ZOOKEEPER=10.10.2.98:2181
      - AUTH_TYPE="LDAP"
      - SPRING_LDAP_URLS="ldap://xxx.xxx.xxx.xxx:389"
      - SPRING_LDAP_BASE="cn={0},ou=xxx,dc=xxx,dc=com"
      - SPRING_LDAP_ADMIN_USER="cn=xxx,dc=xxx,dc=com"
      - SPRING_LDAP_ADMIN_PASSWORD="xxx"
      - SPRING_LDAP_USER_FILTER_SEARCH_BASE="dc=xxx,dc=com"
      - SPRING_LDAP_USER_FILTER_SEARCH_FILTER="(&(uid={0})(objectClass=inetOrgPerson))"
```

## 问题

1. 缺少退出登录功能
2. 缺少中文汉化界面

---

参考文档： 

1. https://www.cnblogs.com/tonglin0325/p/5528560.html
2. https://github.com/provectus/kafka-ui/issues/1466