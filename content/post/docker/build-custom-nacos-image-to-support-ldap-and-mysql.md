---
title: "构建自定义的Nacos镜像支持MySQL数据源与LDAP认证"
date: 2023-03-24T15:59:58+08:00
lastmod: 2023-03-24T15:59:58+08:00
draft: false
keywords: ["nacos","docker","mysql","ldap"]
description: ""
tags: ["docker","nacos","ldap"]
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

`Nacos`官方的`docker`镜像不支持`LDAP`[^1]同时在连接`mysql`方面在某些版本中会出现**No Datasource Set** 的异常，而其官方的release版本则很很稳定，同时又支持`LDAP`。目前部门很多项目都是基于`docker`部署的，处于简化使用，基于部署维护等原因，我决定通过Dockerfile来构建符合自己要求的`nacos`镜像，在确保性能稳定的同时也能支持`ldap`登录。

<!--more-->

## 构建过程

### 初始构建

一开始自己基于[Nacos2.2.1](https://github.com/alibaba/nacos/releases/tag/2.2.1)下载的tar.gz文件进行构建，主要有如下几个步骤：

1. 通过`FROM`拉取`openjdk-8`基础镜像
2. 拷贝并解压`nacos`压缩文件
3. 通过[sed](https://www.gnu.org/software/sed/manual/sed.html)命令来替换`application.properties`中的相关配置
4. 通过`startup.sh -m standalone` 启动`nacos`

初步的`Dockerfile`如下

```dockerfile
FROM openjdk:8-jdk
MAINTAINER 卢运强 "yunqiang.lu@hirain.com"

# 在联网环境下可以通过wget直接下载压缩文件
COPY nacos-server-2.2.1.tar.gz /home/nacos-server-2.2.1.tar.gz
WORKDIR /home

RUN echo "解压nacos文件"
RUN tar -zxvf nacos-server-2.2.1.tar.gz
RUN rm -rf nacos-server-2.2.1.tar.gz nacos/conf/*.sql nacos/conf/*.example nacos/bin/*
COPY startup.sh /home/nacos/bin/startup.sh

RUN mkdir -p /home/nacos/logs


#开始修改配置文件
ARG conf=nacos/conf/application.properties
RUN sed -i "s/server.servlet.contextPath=\/nacos/server.servlet.contextPath=\${SERVER_SERVLET_CONTEXTPATH:\/nacos}/g" $conf
RUN sed -i "s/server.port=8848/server.port=\${NACOS_SERVER_PORT:8848}/g" $conf
#RUN sed -i "s/\#.*spring.datasource.platform=mysql/spring.datasource.platform=\${SPRING_DATASOURCE_PLATFORM:\"mysql\"}/g" $conf
RUN sed -i "s/\#.*spring.sql.init.platform=mysql/spring.sql.init.platform=\${SPRING_DATASOURCE_PLATFORM:mysql}/g" $conf
RUN sed -i "s/\#.*db.num=1/db.num=1/g" $conf
RUN sed -i "s/\#.*db.url.0.*/db.url.0=jdbc:mysql:\/\/\${MYSQL_SERVICE_HOST}:\${MYSQL_SERVICE_PORT:3306}\/\${MYSQL_SERVICE_DB_NAME}\?characterEncoding=utf8\&connectTimeout=1000\&socketTimeout=3000\&autoReconnect=true/g" $conf
RUN sed -i "s/\#.*db.user.0=nacos/db.user=\${MYSQL_SERVICE_USER:root}/g" $conf
RUN sed -i "s/\#.*db.password.0=nacos/db.password=\${MYSQL_SERVICE_PASSWORD}/g" $conf
RUN sed -i "s/.*server.tomcat.accesslog.enabled.*/server.tomcat.accesslog.enabled=\${TOMCAT_ACCESSLOG_ENABLED:false}/g" $conf
RUN sed -i "s/.*nacos.core.auth.plugin.nacos.token.secret.key=.*/nacos.core.auth.plugin.nacos.token.secret.key=SecretKey012345678901234567890123456789012345678901234567890123456789/g" $conf; fi

RUN chmod +x /home/nacos/bin/startup.sh

ENTRYPOINT ["/bin/bash","/home/nacos/bin/startup.sh","-m","standalone"]
```

基于上述文件构建的镜像通过`docker run`指令执行时一直启动不成功，而在`Linux`终端中通过`bash startup.sh -m standlone`的方式则可以顺利启动`nacos`，初次尝试失败!

### 改进版本

 对`startup.sh`进行检查之后，发现其主要是通过如下的`nohup`指令启动的

```ba
if [[ "$JAVA_OPT_EXT_FIX" == "" ]]; then
  nohup "$JAVA" ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
else
  nohup "$JAVA" "$JAVA_OPT_EXT_FIX" ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
fi
```

而`docker`默认支不支持`nohup`[^2],无奈之下只能去`nacos`官网寻找帮助，在其官方GtiHub中找到了一个文档[Dockerfile](https://github.com/nacos-group/nacos-docker/blob/master/build/Dockerfile)，其中的启动脚本为`docker-startup.sh`，对比`startup.sh`发现主要的差异是前者是采用`exec`而非 `nohup`，于是将`Dockerfile`仿照官方说明修改如下，之后能正常启动 

```dockerfile
FROM openjdk:8-jdk
MAINTAINER 卢运强 "lucumt@gmail.com"

RUN echo $JAVA_HOME

#ENV BASE_DIR="/home/nacos"
ENV MODE="standalone" \
    PREFER_HOST_MODE="ip"\
    BASE_DIR="/home/nacos" \
    CLASSPATH=".:/home/nacos/conf:$CLASSPATH" \
    FUNCTION_MODE="all" \
    JAVA_HOME="/usr/local/openjdk-8" \
    NACOS_USER="nacos" \
    JAVA="/usr/local/openjdk-8/bin/java" \
    JVM_XMS="1g" \
    JVM_XMX="1g" \
    JVM_XMN="512m" \
    JVM_MS="128m" \
    JVM_MMS="320m" \
    NACOS_DEBUG="y" \
    TOMCAT_ACCESSLOG_ENABLED="false" \
    TIME_ZONE="Asia/Shanghai"

# 在联网环境下可以通过wget直接下载压缩文件
COPY nacos-server-2.2.1.tar.gz /home/nacos-server-2.2.1.tar.gz
WORKDIR /home

RUN echo "解压nacos文件"
RUN tar -zxvf nacos-server-2.2.1.tar.gz
RUN rm -rf nacos-server-2.2.1.tar.gz nacos/conf/*.sql nacos/conf/*.example nacos/bin/*
COPY docker-startup.sh /home/nacos/bin/docker-startup.sh

RUN mkdir -p /home/nacos/logs


#开始修改配置文件
ARG conf=nacos/conf/application.properties
RUN sed -i "s/server.servlet.contextPath=\/nacos/server.servlet.contextPath=\${SERVER_SERVLET_CONTEXTPATH:\/nacos}/g" $conf
RUN sed -i "s/server.port=8848/server.port=\${NACOS_SERVER_PORT:8848}/g" $conf
#RUN sed -i "s/\#.*spring.datasource.platform=mysql/spring.datasource.platform=\${SPRING_DATASOURCE_PLATFORM:\"mysql\"}/g" $conf
RUN sed -i "s/\#.*spring.sql.init.platform=mysql/spring.sql.init.platform=\${SPRING_DATASOURCE_PLATFORM:mysql}/g" $conf
RUN sed -i "s/\#.*db.num=1/db.num=1/g" $conf
RUN sed -i "s/\#.*db.url.0.*/db.url.0=jdbc:mysql:\/\/\${MYSQL_SERVICE_HOST}:\${MYSQL_SERVICE_PORT:3306}\/\${MYSQL_SERVICE_DB_NAME}\?characterEncoding=utf8\&connectTimeout=1000\&socketTimeout=3000\&autoReconnect=true/g" $conf
RUN sed -i "s/\#.*db.user.0=nacos/db.user=\${MYSQL_SERVICE_USER:root}/g" $conf
RUN sed -i "s/\#.*db.password.0=nacos/db.password=\${MYSQL_SERVICE_PASSWORD}/g" $conf
RUN sed -i "s/.*server.tomcat.accesslog.enabled.*/server.tomcat.accesslog.enabled=\${TOMCAT_ACCESSLOG_ENABLED:false}/g" $conf
RUN sed -i "s/.*nacos.core.auth.plugin.nacos.token.secret.key=.*/nacos.core.auth.plugin.nacos.token.secret.key=SecretKey012345678901234567890123456789012345678901234567890123456789/g" $conf

RUN chmod +x /home/nacos/bin/docker-startup.sh

ENTRYPOINT ["/bin/bash","/home/nacos/bin/docker-startup.sh","-m","standalone"]
```

### 支持LDAP

结合公司的实际情况，在部门内部使用时一般采用`LDAP`登录，而交付给客户时更多的采用普通账号登录，为此需要2份`Dockerfile`来构建2个不同的镜像，处于简化维护的考虑，自己决定采用1份`Dockerfile`文件根据构建参数来动态的生成不同的镜像。

在网络上搜索后发现可以在`Dockerfile`中执行类似if else的指令[^3]，于是在原有的`Dockerfile`基础上添加如下指令即可动态的支持`LDAP`登录

```dockerfile
ARG LOGIN_TYPE=nacos
RUN sed -i "s/nacos.core.auth.system.type=.*/nacos.core.auth.system.type=${LOGIN_TYPE}/g" $conf
RUN if [ $LOGIN_TYPE = "nacos" ];then echo "基于普通登录方式构建";else echo "基于ldap登录方式构建"; fi

# ldap登录认证方式的额外处理
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.url=.*/nacos.core.auth.ldap.url=\${LDAP_URL}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.basedc=.*/nacos.core.auth.ldap.basedc=\${LDAP_BASE_DC}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.userDn=.*/nacos.core.auth.ldap.userDn=\${LDAP_USER_DN}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.password=.*/nacos.core.auth.ldap.password=\${LDAP_USER_PASSWORD}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.filter.prefix=.*/nacos.core.auth.ldap.filter.prefix=\${LDAP_UID}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.case.sensitive=.*/nacos.core.auth.ldap.case.sensitive\${LDAP_CASE_SENSITIVE}/g" $conf; fi
```

## 最终文件 

### Dockerfile

```dockerfile
FROM openjdk:8-jdk
MAINTAINER 卢运强 "lucumt@gmail.com"

RUN echo $JAVA_HOME

#ENV BASE_DIR="/home/nacos"
ENV MODE="standalone" \
    PREFER_HOST_MODE="ip"\
    BASE_DIR="/home/nacos" \
    CLASSPATH=".:/home/nacos/conf:$CLASSPATH" \
    FUNCTION_MODE="all" \
    JAVA_HOME="/usr/local/openjdk-8" \
    NACOS_USER="nacos" \
    JAVA="/usr/local/openjdk-8/bin/java" \
    JVM_XMS="1g" \
    JVM_XMX="1g" \
    JVM_XMN="512m" \
    JVM_MS="128m" \
    JVM_MMS="320m" \
    NACOS_DEBUG="y" \
    TOMCAT_ACCESSLOG_ENABLED="false" \
    TIME_ZONE="Asia/Shanghai"

# 在联网环境下可以通过wget直接下载压缩文件
COPY nacos-server-2.2.1.tar.gz /home/nacos-server-2.2.1.tar.gz
WORKDIR /home

RUN echo "解压nacos文件"
RUN tar -zxvf nacos-server-2.2.1.tar.gz
RUN rm -rf nacos-server-2.2.1.tar.gz nacos/conf/*.sql nacos/conf/*.example nacos/bin/*
COPY docker-startup.sh /home/nacos/bin/docker-startup.sh

RUN mkdir -p /home/nacos/logs


#开始修改配置文件
ARG conf=nacos/conf/application.properties
RUN sed -i "s/server.servlet.contextPath=\/nacos/server.servlet.contextPath=\${SERVER_SERVLET_CONTEXTPATH:\/nacos}/g" $conf
RUN sed -i "s/server.port=8848/server.port=\${NACOS_SERVER_PORT:8848}/g" $conf
#RUN sed -i "s/\#.*spring.datasource.platform=mysql/spring.datasource.platform=\${SPRING_DATASOURCE_PLATFORM:\"mysql\"}/g" $conf
RUN sed -i "s/\#.*spring.sql.init.platform=mysql/spring.sql.init.platform=\${SPRING_DATASOURCE_PLATFORM:mysql}/g" $conf
RUN sed -i "s/\#.*db.num=1/db.num=1/g" $conf
RUN sed -i "s/\#.*db.url.0.*/db.url.0=jdbc:mysql:\/\/\${MYSQL_SERVICE_HOST}:\${MYSQL_SERVICE_PORT:3306}\/\${MYSQL_SERVICE_DB_NAME}\?characterEncoding=utf8\&connectTimeout=1000\&socketTimeout=3000\&autoReconnect=true/g" $conf
RUN sed -i "s/\#.*db.user.0=nacos/db.user=\${MYSQL_SERVICE_USER:root}/g" $conf
RUN sed -i "s/\#.*db.password.0=nacos/db.password=\${MYSQL_SERVICE_PASSWORD}/g" $conf
RUN sed -i "s/.*server.tomcat.accesslog.enabled.*/server.tomcat.accesslog.enabled=\${TOMCAT_ACCESSLOG_ENABLED:false}/g" $conf
RUN sed -i "s/.*nacos.core.auth.plugin.nacos.token.secret.key=.*/nacos.core.auth.plugin.nacos.token.secret.key=SecretKey012345678901234567890123456789012345678901234567890123456789/g" $conf

ARG LOGIN_TYPE=nacos
RUN sed -i "s/nacos.core.auth.system.type=.*/nacos.core.auth.system.type=${LOGIN_TYPE}/g" $conf
RUN if [ $LOGIN_TYPE = "nacos" ];then echo "基于普通登录方式构建";else echo "基于ldap登录方式构建"; fi

# ldap登录认证方式的额外处理
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.url=.*/nacos.core.auth.ldap.url=\${LDAP_URL}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.basedc=.*/nacos.core.auth.ldap.basedc=\${LDAP_BASE_DC}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.userDn=.*/nacos.core.auth.ldap.userDn=\${LDAP_USER_DN}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.password=.*/nacos.core.auth.ldap.password=\${LDAP_USER_PASSWORD}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.filter.prefix=.*/nacos.core.auth.ldap.filter.prefix=\${LDAP_UID}/g" $conf; fi
RUN if [ $LOGIN_TYPE = "ldap" ];then sed -i "s/\#nacos.core.auth.ldap.case.sensitive=.*/nacos.core.auth.ldap.case.sensitive\${LDAP_CASE_SENSITIVE}/g" $conf; fi

RUN chmod +x /home/nacos/bin/docker-startup.sh

ENTRYPOINT ["/bin/bash","/home/nacos/bin/docker-startup.sh","-m","standalone"]
```

### 构建方式

在构建时需要将`Dockerfile`、`docker-startup.sh`以及对应的`nacos`压缩文件放到同一个目录下

* 普通登录方式构建

  ```bash
  # 不传递登录参数
  docker build -t nacos_custom:v1.0 .
  
  # 显示指定登录参数
  docker build -t nacos_custom:v1.0 --build-arg LOGIN_TYPE=nacos .
  ```

* `LDAP`登录方式构建

  ```bash
  docker build -t nacos_custom:v1.0 --build-arg LOGIN_TYPE=ldap .
  ```

### 使用方式

假设存储数据库为`mysql`采用[docker-compose](https://docs.docker.com/compose/)方式登录，相关的`docker-compose.yml`文件如下:

* 普通方式登录

  ```yaml
  version: "3"
  services:
    nacos:
      image: nacos_custom:v1.0
      restart: always
      container_name: nacos_custom
      ports:
        - 8858:8858
      environment:
        - TZ=Asia/Shanghai
        - NACOS_SERVER_PORT=8858
        - SPRING_DATASOURCE_PLATFORM=mysql
        - MYSQL_SERVICE_HOST=xxxx
        - MYSQL_SERVICE_PORT=xxxx
        - MYSQL_SERVICE_DB_NAME=nacos_test
        - MYSQL_SERVICE_USER=root
        - MYSQL_SERVICE_PASSWORD=654321
      volumes:
        - $PWD/logs:/home/nacos/logs/
  ```

* `LDAP`登录

  ```yaml
  version: "3"
  services:
    nacos:
      image: nacos_custom:v1.0
      restart: always
      container_name: nacos_custom
      ports:
        - 8858:8858
      environment:
        - TZ=Asia/Shanghai
        - NACOS_SERVER_PORT=8858
        - SPRING_DATASOURCE_PLATFORM=mysql
        - MYSQL_SERVICE_HOST=xxxx
        - MYSQL_SERVICE_PORT=3316
        - MYSQL_SERVICE_DB_NAME=nacos_test
        - MYSQL_SERVICE_USER=root
        - MYSQL_SERVICE_PASSWORD=xxxx
        - LDAP_URL=ldap://xxxx:389
        - LDAP_BASE_DC=dc=xxx,dc=xxx
        - LDAP_USER_DN=cn=xxx,dc=xxx,dc=com
        - LDAP_USER_PASSWORD=xxxx
        - LDAP_UID=uid
        - LDAP_CASE_SENSITIVE=false
      volumes:
        - $PWD/logs:/home/nacos/logs/
  ```

### 镜像参数说明

| 属性                           | 作用                     | 默认值 | 可选值      |
| ------------------------------ | ------------------------ | ------ | ----------- |
| **NACOS_SERVER_PORT**          | nacos服务器端口          | 8848   |             |
| **SPRING_DATASOURCE_PLATFORM** | 指定nacos的数据源        | mysql  | `mysql`或空 |
| **MYSQL_SERVICE_HOST**         | mysql服务器地址          |        |             |
| **MYSQL_SERVICE_PORT**         | mysql服务器端口          | 3306   |             |
| **MYSQL_SERVICE_DB_NAME**      | mysql数据库名称          |        |             |
| **MYSQL_SERVICE_USER**         | mysql数据库用户名        | root   |             |
| **MYSQL_SERVICE_PASSWORD**     | mysql数据据密码          |        |             |
| **LDAP_URL**                   | ldap服务的地址和端口号   |        |             |
| **LDAP_BASE_DC**               | ldap搜索范围             |        |             |
| **LDAP_USER_DN**               | ldap绑定账号[^4]         |        |             |
| **LDAP_USER_PASSWORD**         | ldap绑定账号的密码       |        |             |
| **LDAP_UID**                   | 用户账号字段             |        |             |
| **LDAP_CASE_SENSITIVE**        | ldap认证时是否大小写敏感 |        |             |

[^1]: https://github.com/alibaba/nacos/issues/9751
[^2]:https://unix.stackexchange.com/questions/268284/nohup-doesnt-work-as-expected-in-docker-script
[^3]:https://stackoverflow.com/questions/43654656/dockerfile-if-else-condition-with-external-arguments
[^4]: 部分`LDAP`数据库不支持匿名登录，此时需要管理员账号来绑定登录