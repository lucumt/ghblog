---
title: "多个docker-compose.yml文件启动时容器冲突的原因分析"
date: 2022-07-25T22:53:33+08:00
lastmod: 2022-07-25T22:53:33+08:00
draft: false
keywords: ["docker","docker-compose","project","conflict"]
description: "简要记录在同一个文件夹下多个docker-compose.yml文件分别启动时导致的容器冲突原因分析"
tags: ["docker"]
categories: ["容器化"]
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

简要记录下近期在项目中遇到的多个`docker-compose.yml`文件容器启动时发生冲突的原因分析。

<!--more-->

## 问题背景

出于多方面的考虑，公司的一些项目都逐渐的采用[**Docker**](https://www.docker.com/)作为部署环境，相对于互联网公司我们的`Docker`容器数据量没那么多，在容器管理工具的选型上我们采用的是[**Docker Compose**](https://docs.docker.com/compose/)而非常见的[**Kubernetes**](https://kubernetes.io/)。

`Docker Compose`需要我们将相关的`Docker`指令都写入到`yaml`文件中，相对于通过纯命令行操作`Docker`其在可维护性和可阅读性上都有很大的便利，故很快在部门内部推广使用了。

在此过程中，有同事偶然遇到多个`docker-compose.yml`文件启动时会冲突的问题，一番分析后虽然最终解决该问题，但由于该问题在多个`docker-compose.yml`文件时容易产生，简单记录下已被不时之需。

简化版的问题描述如下：

1. 在某个目录下有两个`docker-compose.yml`文件，分别为`nginx_test_1.yml`和`nginx_test_2.yml`

   ```bash
   [root@fox docker_test]# pwd
   /root/docker_test
   [root@fox docker_test]# ls
   nginx_test_1.yml  nginx_test_2.yml
   ```

2. 它们的内容分别如下

   `nginx_test_1.yml`配置：

   ```yaml
   version: "3"
   services:
      nginx:
        privileged: true
        image: nginx
        restart: always
        container_name: nginx_test_1
        ports:
          - "8081:80"
   ```

   `nginx_test_2.yml`配置：

   ```yaml
   version: "3"
   services:
      nginx:
        privileged: true
        image: nginx
        restart: always
        container_name: nginx_test_2
        ports:
          - "8082:80"
   ```

3. 基于`nginx_test_1.yml`文件可正常启动

   ![docker compose正常启动](/blog_img/docker/multiple-docker-compose-conflict-analysis/docker-compose-start-success.png  "docker compose正常启动")

4. 基于`nginx_test_1.yml`文件进行测试时，虽然也能启动，但结果不符合我们的预期，其把名为`nginx_test_1`的容器关闭了，而我们期望的是基于两个文件启动时，能分别启动不同的容器

   ![docker compose未按预期启动](/blog_img/docker/multiple-docker-compose-conflict-analysis/docker-compose-start-not-expected.png  "docker compose未按预期启动")

5. 利用`docker ps -a`查询也只有1个容器

   ![docker ps查询只有1个容器](/blog_img/docker/multiple-docker-compose-conflict-analysis/docker-ps-query-1.png  "docker ps查询只有1个容器")

发生了什么？ 理论上不应该是采用不同的文件可分别正常启动么？！

## 原因分析

一开始自己以为是`docker-compose.yml`文件的问题，以为有语法问题，反复对这两个文件进行检查后并没有找出啥。

同时`nginx_test_1.yml`和`nginx_test_2.yml`按照下述的指令操作，可正常启动：

```bash
docker-compose -f nginx_test_1.yml up -d
docker ps
docker-compose -f nginx_test_1.yml down
docker-compose -f nginx_test_2.yml up -d
docker ps
docker-compose -f nginx_test_1.yml down
```

若按照之前的方式操作依旧有问题，至此可以确认的是单个`docker-compose.yml`使用时没有问题，多个`docker-compose.yml`文件一起使用时会有问题。

需要从`docker-compos.yml`的使用机制着手。

在网上搜索后，发现有人已经遇到类似的问题[**Docker is not creating new container but recreates running one**](https://stackoverflow.com/questions/43126022/docker-is-not-creating-new-container-but-recreates-running-one)，通过对该文章的阅读发现了问题根源：

> `Docker Compose`通过项目名称和服务名称的组合来识别一个指定的服务 ，启动和关闭时基于该组合进行操作。
>
> <br>
>
> 其中服务名称是在`yaml`文件中指定的，而服务名称可通过启动时`-p`参数指定，若没有该参数，则去环境变量中查找`COMPOSE_PROJECT_NAME`的值，若还是没有则以当前`yaml`文件的文件夹作为项目名称。

具体到本问题中，在启动时没有指定`-p`参数，也没有指定`COMPOSE_PROJECT_NAME`环境变量，同时这两个`yaml`文件都位于`docker_test`目录下，则项目名称为`docker_test`，查看其源码，发现其服务名称都为`nginx`，由此导致问题产生！

```yaml
version: "3"
services:
   # 服务名称都为nginx
   nginx:
     privileged: true
     image: nginx
     restart: always
     container_name: nginx_test_1
     ports:
       - "8081:80"
```

## 解决方案

在`Docker Compose`的官网找到关于`COMPOSE_PROJECT_NAME`的[**说明**](https://docs.docker.com/compose/environment-variables/envvars/#compose_project_name)如下

![COMPOSE_PROJECT_NAME配置说明](/blog_img/docker/multiple-docker-compose-conflict-analysis/compose_project_name_instruction.png  "COMPOSE_PROJECT_NAME配置说明")

上述说明前半部分说明了`Docker Compose`会将项目名和服务名组合作为容器名称供启动使用，基于该描述前述两个`yaml`文件理论上的容器名称都为`docker_test_nginx-1`[^1]，故而导致启动时冲突。

查看本文开始的截图，发现容器名并不是`docker_test_nginx`，其原因为我们在`yaml`文件中手工指定了`container_name`，若取消该配置重新启动，则会发现容器名称已经变为`docker_test_nginx_1`:

![docker compose自动生成容器名](/blog_img/docker/multiple-docker-compose-conflict-analysis/docker-compose-default-container-name.png  "docker compose自动生成容器名")



后半部分说明项目名称的计算规则，按照优先级从高到低如下：

1. 启动容器时，在命令行中通过`-p`指定
2. 基于环境变量`COMPOSE_PROJECT_NAME`获取
3. 基于`docker-compose.yml`文件最顶层的`name`属性获取，有多个`yaml`文件时以最后一个为准
4. 包含要当前运行的`yaml`文件的最底层文件夹名称
5. 当前要运行的`yaml`文件所在的文件夹名称

其中4和5的表达的意思基本一样，主要是为是否通过文件夹指定`yaml`文件。

如通过`docker-compose -f a/b/nginx_test_1.yml up -d`来执行时，基于规则4其容器名称为`Creating c_nginx_1 `

![docker compose在有文件夹时自动生成容器名](/blog_img/docker/multiple-docker-compose-conflict-analysis/docker-compose-default-container-name-in-folders.png  "docker compose在有文件夹时自动生成容器名")

而通过`docker-compose -f nginx_test_1.yml up -d`来执行时，则是基于规则5来指定容器名。



接下来演示通过前述规则解决冲突。

* 基于命令行参数

  ![docker compose通过命令行指定项目名](/blog_img/docker/multiple-docker-compose-conflict-analysis/docker-compose-run-via-command-line-project.png  "docker compose通过命令行指定项目名")

* 基于不同文件夹

  ![docker compose通过不同文件夹启动](/blog_img/docker/multiple-docker-compose-conflict-analysis/docker-compose-run-via-different-folders.png  "docker compose通过不同文件夹启动")

* 基于`name`属性，个人在`v2.26.1`版本上测试通过

  ![docker compose通过name属性启动](/blog_img/docker/multiple-docker-compose-conflict-analysis/docker-compose-run-via-name-property.png  "docker compose通过name属性启动")

* 基于`COMPOSE_PROJECT_NAME`环境变量，个人在`v2.2.2`版本上测试通过

  ![docker compose通过环境变量启动](/blog_img/docker/multiple-docker-compose-conflict-analysis/docker-compose-run-via-env-files.png  "docker compose通过环境变量启动")

[^1]: 其中的`-1`是`Docker Compose`自动添加的序号