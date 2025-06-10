---
title: "利用Nginx来代理MinIO公共桶和私有桶的访问URL"
date: 2025-06-10T09:40:38+08:00
lastmod: 2025-06-10T09:40:38+08:00
draft: true
keywords: []
description: ""
tags: ["minio"]
categories: []
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

<!--more-->

参考[这篇文章](https://www.cnblogs.com/zoujiaojiao/p/18534444)

`Nginx`相关配置如下

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

