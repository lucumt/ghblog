---
title: "利用Nginx来代理MinIO私有桶的分享链接地址"
date: 2025-06-10T09:40:38+08:00
lastmod: 2025-06-10T09:40:38+08:00
draft: true
keywords: []
description: ""
tags: ["minio"]
categories: ["网络编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
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
codeTabSeperator: "::"
moreMeta: true

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

简要介绍通过如何通过`Nginx`实现将`MinIO`中文件的请求地址进行代理转发，实现不同网络环境的隔离。

<!--more-->

# 问题背景

部门项目需要部署在公网，该项目中使用到了`MinIO`作为文件存储，出于安全考虑it部门只开放了一个`443`端口。

我们的项目是前后端分离的架构，且对于avatar头像，富文本编辑中的图片，用户上传的文件等各类型文件均通过`MinIO`进行存储，之前在内部网络中都是直接访问`MinIO`生成的URL访问链接。而现在的公网环境只有一个端口可用，前端页面访问、`MinIO`文件访问等都必须共用这一个宝贵的端口。

前后端共用可用类似如下代码实现

```nginx
server {
    listen       8080;
    server_name  app-web;

    # 前端页面访问入口
    location / {
      root   /usr/share/nginx/html/app-web;
      index  index.html index.htm;
          try_files $uri $uri/ /index.html;
    }

    # 后端API代理配置
    location /api/ {
          proxy_pass http://192.168.40.84:13001;
          rewrite ^/api/(.*) /$1 break;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $realip_remote_addr;
          proxy_set_header REMOTE-HOST $realip_remote_addr;
    }
}
```

自己想到了如下几种方案：

1. 将`MinIO`设置为公开桶，然后通过`Nginx`进行代理，此种方式和前述的类似，且公开桶即使在局域网也有风险
2. 在`Nginx`中收到请求后利用`Lua`脚本给`MinIO`发送请求然后返回
3. 放弃直接使用超链接的方案，所有的文件预览与下载请求均通过给后端发送请求，后端`SpringBoot`接收到请求后通过代码的方式访问`MinIO`

# 解决方案

参考[这篇文章](https://www.cnblogs.com/zoujiaojiao/p/18534444)


{{< details "点击查看Nginx完整配置代码" >}}
```nginx
server {
    listen       8080;
    server_name  app-web;

    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST,PUT,DELETE, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
    add_header Cache-Control 'no-cache';

    client_max_body_size  2048M;

    location / {
      root   /usr/share/nginx/html/app-web;
      index  index.html index.htm;
          try_files $uri $uri/ /index.html;
    }

    # 后端API代理配置
    location /api/ {
          proxy_pass http://192.168.40.84:13001;
          rewrite ^/api/(.*) /$1 break;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $realip_remote_addr;
          proxy_set_header REMOTE-HOST $realip_remote_addr;
          #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # 后端API代理配置
    location /file/{
        proxy_pass http://192.168.40.84:9200/;
        rewrite ^/file/(.*) /$1 break;
        proxy_set_header Host $proxy_host;  #私有桶需要使用$proxy_host
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 确保传递与签名验证相关的头部
        proxy_set_header X-Amz-Algorithm $http_x_amz_algorithm;
        proxy_set_header X-Amz-Credential $http_x_amz_credential;
        proxy_set_header X-Amz-Date $http_x_amz_date;
        proxy_set_header X-Amz-Expires $http_x_amz_expires;
        proxy_set_header X-Amz-SignedHeaders $http_x_amz_signedheaders;
        proxy_set_header X-Amz-Signature $http_x_amz_signature;

        # 保持连接
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_buffering off;
        client_max_body_size 0;
   }


    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
{{< /details >}}

