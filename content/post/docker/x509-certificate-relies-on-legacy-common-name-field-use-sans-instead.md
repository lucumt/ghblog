---
title: "在Docker中遇到x509: certificate relies on legacy Common Name field, use SANs instead问题的解决"
date: 2023-07-06T09:54:01+08:00
lastmod: 2023-07-06T09:54:01+08:00
draft: false
keywords: ["docker","kubernetes","kubesphere","x509"]
description: "在KubeSphere使用过程中遇到x509: certificate relies on legacy Common Name field, use SANs instead问题的解决，简要介绍其解决方案"
tags: ["docker","kubernetes","kubesphere"]
categories: ["持续集成","工具使用"]
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

近期在给公司内部安装`KubeSphere`新环境和维护原有`KubeSphere`环境的过程中，频繁遇到`x509: certificate relies on legacy Common Name field, use SANs instead`，此问题会影响正常功能的使用，简要介绍其解决方案。

<!--more-->

## 问题描述

本操作过程基于[KubeSphere离线安装](https://www.kubesphere.io/zh/docs/v3.3/installing-on-kubernetes/on-prem-kubernetes/install-ks-on-linux-airgapped/)一文来操作的。

1. 执行下述命令生成自己的证书

   ```bash
   mkdir -p certs
   
   openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 36500 -out certs/domain.crt
   ```

2. 在生成证书的过程中按照文档要求将`Common Name`的值设置为`dockerhub.kubekey.local`

   ![生成自定义证书](/blog_img/docker/x509-certificate-relies-on-legacy-common-name-field-use-sans-instead/generate-custom-cert.png "生成自定义证书") 

3. 在`/etc/hosts`中将`dockerhub.kubekey.local`映射到当前服务器IP地址

   ```
   192.168.0.2 dockerhub.kubekey.local
   ```

4. 让`Docker`信任刚生成的证书

   ```bash
   mkdir -p  /etc/docker/certs.d/dockerhub.kubekey.local
   cp certs/domain.crt  /etc/docker/certs.d/dockerhub.kubekey.local/ca.crt
   ```

5. 执行下述命令启动`Docker`仓库

   ```bash
   docker run -d \
     --restart=always \
     --name registry \
     -v "$(pwd)"/certs:/certs \
     -v /mnt/registry:/var/lib/registry \
     -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
     -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
     -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
     -p 443:443 \
     registry:2
   ```

6. 在终端执行`docker pull dockerhub.kubekey.local/nginx`，运行结果如下，提示原证书的配置方式过期，不能正常使用，问题出现!

   ![由于证书问题导致本地仓库无法工作](/blog_img/docker/x509-certificate-relies-on-legacy-common-name-field-use-sans-instead/docker-registry-cert-validate-failed.png "由于证书问题导致本地仓库无法工作") 

## 分析与解决

1. 在[https://go.dev/doc/go1.15#commonname](https://go.dev/doc/go1.15#commonname)中找到如下一段说明

   > The deprecated, legacy behavior of treating the `CommonName` field on X.509 certificates as a host name when no Subject Alternative Names are present is now disabled by default. It can be temporarily re-enabled by adding the value `x509ignoreCN=0` to the `GODEBUG` environment variable.
   >
   > Note that if the `CommonName` is an invalid host name, it's always ignored, regardless of `GODEBUG` settings. Invalid names include those with any characters other than letters, digits, hyphens and underscores, and those with empty labels or trailing dots

   从中可知在`Go10.15`之后`Common Name`这个字段已经废弃，可在`GODEBUG`中通过配置`x509ignoreCN=0`来重新启用此字段，不过我们是基于`Go`的`Docker`应用环境，而非`Go`开发环境，显然此种方式不大可行。

2. 继续搜索，找到这篇文章[how-do-i-use-sans-with-openssl-instead-of-common-name](https://stackoverflow.com/questions/64814173/how-do-i-use-sans-with-openssl-instead-of-common-name)，其中提供了一个解决思路

   ![Stackoverflow解决方案](/blog_img/docker/x509-certificate-relies-on-legacy-common-name-field-use-sans-instead/stackoverflow-solution.png "Stackoverflow解决方案") 

3. 对原有生成证书的步骤改进如下，然后重新执行生成证书

   ```bash
   openssl req  -addext "subjectAltName = DNS:dockerhub.kubekey.local" -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 36500 -out certs/domain.crt
   ```

4. 重新执行`docker pull dockerhub.kubekey.local/nginx`可发现证书问题已经解决

   ![证书问题解决](/blog_img/docker/x509-certificate-relies-on-legacy-common-name-field-use-sans-instead/docker-registry-cert-validate-success.png "证书问题解决") 

5. 前述执行结果任然报错的原因为镜像不存在，切换为一个已经存在的镜像重新执行即可

   ![docker可正常拉取镜像](/blog_img/docker/x509-certificate-relies-on-legacy-common-name-field-use-sans-instead/docker-registry-pull-success.png "docker可正常拉取镜像") 