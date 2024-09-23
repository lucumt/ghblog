---
title: "基于Docker创建Harbor镜像仓库系统"
date: 2021-05-25T10:05:48+08:00
lastmod: 2021-05-25T10:05:48+08:00
draft: false
keywords: ["docker","harbor"]
description: "以图形化的方式描述如何基于Docker创建Harbor镜像仓库系统，给相关使用人员提供参考"
tags: ["docker","harbor","devops"]
categories: ["容器化","持续集成"]
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

简要介绍基于`docker-compose`安装`Harbor`的过程，供后续参考。

<!--more-->

1. 下载`Harbor`的离线安装文件并解压

   ```bash
   wget https://github.com/goharbor/harbor/releases/download/v2.1.4/harbor-offline-installer-v2.1.4.tgz
   
   # 解压后会生成名称harbor的目录
   tar xvf harbor-offline-installer-v2.1.4.tgz
   ```

2. 进入harbor目录下可发现其有对应的文件

   ```bash
   root@test:~/lyq# cd harbor
   root@test:~/lyq/harbor# ls
   common.sh  harbor.v2.1.4.tar.gz  harbor.yml.tmpl  install
   ```

3. 在harbor根目录下执行下述命令生成配置文件

   ```bash
   cp harbor.yml.tmpl harbor.yml
   ```

4. 修改前一个步骤中生成的`harbor.yml`，出于简化考虑屏蔽掉`HTTPS`相关的配置，根据实际情况修改其对外端口和密码

   ![harbor配置文件修改](/blog_img/docker/install-harbor-via-docker/harbor-yml-config.png "harbor配置文件修改") 

5. 运行`bash install.sh`开启安装过程，若第一次安装，则该脚本会首先下载对应的镜像，之后才会进行真正的安装过程

   ![harbor安装过程](/blog_img/docker/install-harbor-via-docker/harbor-install-process.png "harbor安装过程") 

6. 若安装过程中一切正常，则最后会有类似如下输出

   ![harbor安装结果](/blog_img/docker/install-harbor-via-docker/harbor-install-result.png "harbor安装结果") 

7. 再次查看该目录下的文件，发现多了一个名为`docker-compose.yml`的文件，此文件是由安装脚本自动生成的

   ```bash
   root@test:~/lyq/harbor# ls
   common  common.sh  docker-compose.yml  harbor.v2.1.4.tar.gz  harbor.yml  harbor.yml.tmpl  install.sh  LICENSE  prepare
   ```

8. 输入`docker-compose ps`可查看`Harbor`相关的`Docker`容器状态

   ![harbor容器检查](/blog_img/docker/install-harbor-via-docker/harbor-container-check.png "harbor容器检查") 

9. 基于`harbor.yml`中的配置，可知其访问地址为`http://10.0.8.147:8087 `，在浏览器中输入该地址，可正常打开

   ![harbor登录主页](/blog_img/docker/install-harbor-via-docker/harbor-login-page.png "harbor登录主页")

10. 使用默认配置的账号密码`admin/Harbor12345`登录系统，显示如下，至此`Harbor`安装完毕！

    ![harbor系统主页](/blog_img/docker/install-harbor-via-docker/harbor-home-page.png "harbor系统主页")

11. 若要停止或重启`Harbor`，可在该目录下执行如下指令

    ```bash
    # 停止harbor
    docker-compose down
    
    # 启动harbor
    docker-compose up -d
    
    # 重启harbor
    docker-compose restart
    ```

    