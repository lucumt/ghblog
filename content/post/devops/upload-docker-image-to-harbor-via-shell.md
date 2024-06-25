---
title: "利用Linux脚本将Docker镜像手工上传到Harbor仓库"
date: 2021-06-24T20:38:59+08:00
lastmod: 2021-06-24T20:38:59+08:00
draft: false
keywords: ["docker","docker images","harbor"]
description: "简要介绍如何利用Linux脚本将Docker镜像手工上传到Harbor仓库"
tags: ["docker","harbor"]
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

简要说明如何通过`Linux`指令将特定的`Docker`镜像手工上传到`Harbor`仓库

<!--more-->

1. `Harbor`系统的访问地址为`http://aeectss.xxxx.local:30005/`，在要操作的电脑上确保已经给`/etc/docker/daemon.json`添加了`insecure-registries`的配置，使得`Harbor`在非HTTPS协议下也能上传

   ```json
   {
     "insecure-registries": ["aeectss.xxxx.local:30005"],
   }
   ```

   若没有上述配置，在添加完毕之后需要通过`systemctl daemon-reload && systemctl restart docker`让修改生效

2. 确保要操作人员的账户在`harbor`中对应的项目下具有`项目管理员`权限，类似如下图所示

   ![harbor项目给用户授权](/blog_img/devops/upload-docker-image-to-harbor-via-shell/harbor-project-user-authentication.png "harbor项目给用户授权")

3. 在对应电脑的终端上执行下述指令，此处的账号为前一个步骤配置好的具有权限的账号

   ```bash
   image=orienlink-frame-extraction:1.3.4.3 \
   && docker login -u yunqaing.lu -p xxxx aeectss.xxxx.local:30005  \
   && docker tag  orienlink-frame-extraction:1.3.4.4 aeectss.xxxx.local:30005/orienlink-product-library/$image  \
   && docker push aeectss.xxxx.local:30005/orienlink-product-library/$image  \
   && docker rmi aeectss.xxxx.local:30005/orienlink-product-library/$image  \
   && docker logout aeectss.xxxx.local:30005
   ```

   或将上述命令修改为更灵活的shell脚本，类似如下：

   ```bash
   #!/bin/bash
   printf "请输入对应的用户名:"
   read username
   
   read -s -p "请输入对应的密码:" password
   
   printf "\n请输入要上传的镜像与版本:"
   read tag
   
   docker login -u $username -p $password aeectss.xxxx.local:30005
   docker tag $tag aeectss.xxxx.local:30005/orienlink-product-library/$tag
   docker push aeectss.xxxx.local:30005/orienlink-product-library/$tag
   docker rmi aeectss.xxxx.local:30005/orienlink-product-library/$tag
   docker logout aeectss.xxxx.local:30005
   ```

4. 若一切正常，执行结果类似如下所示

   ![docker镜像上传结果](/blog_img/devops/upload-docker-image-to-harbor-via-shell/docker-image-push-result.png "docker镜像上传结果")

5. 去`harbor`仓库检测后可发现对应的镜像已经上传成功

   ![harbor中项目镜像上传成功](/blog_img/devops/upload-docker-image-to-harbor-via-shell/harbor-image-push-result.png "harbor中项目镜像上传成功")

6. 若重复上传同一个`tag`的镜像，则第二次上传会较快。
