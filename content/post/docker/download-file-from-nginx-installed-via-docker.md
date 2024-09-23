---
title: "在利用docker安装的nginx中实现静态文件下载"
date: 2021-03-23T10:46:12+08:00
lastmod: 2021-03-23T10:46:12+08:00
draft: false
keywords: ["docker","nginx","静态文件"]
description: "简要介绍利用docker安装的nginx中实现静态文件下载,实现快速的文件下载功能"
tags: ["docker","nginx"]
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

---

简单记录利用`nginx`结合`docker`实现一个简易的静态文件下载服务器。

<!--more-->

* `docker-compose.yml`配置

  ```yaml
  version: "3"
  services:
     nginx:
       privileged: true
       image: nginx
       restart: always
       container_name: nginx_file_server
       environment:
        - "TZ=Asia/Shanghai"
       ports:
         - "8788:8788"
       volumes:
         - $PWD/html:/usr/share/nginx/html
         - $PWD/conf.d:/etc/nginx/conf.d
         - $PWD/nginx.conf:/etc/nginx/nginx.conf
         - $PWD/logs:/var/log/nginx
  ```

* `nginx.conf`配置

  ```nginx
  user  nginx;
  worker_processes  auto;
  
  error_log  /var/log/nginx/error.log notice;
  pid        /var/run/nginx.pid;
  
  
  events {
      worker_connections  1024;
  }
  
  
  http {
      include       /etc/nginx/mime.types;
      default_type  application/octet-stream;
  
      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
  
      access_log  /var/log/nginx/access.log  main;
  
      sendfile        on;
      #tcp_nopush     on;
  
      keepalive_timeout  65;
  
      #gzip  on;
  
      include /etc/nginx/conf.d/*.conf;
  }
  ```

* `conf.d/file.conf`配置

  ```nginx
  
  server {
          listen       8788;
          server_name  zip-download;
  
          location ~.*\.(zip)$ {
                  root /usr/share/nginx/html/;
                  # 自动创建目录文件列表为首页
                  autoindex on;
                  # 自动首页的格式为html
                  autoindex_format html;
                  # 关闭文件大小转换
                  autoindex_exact_size off;
                  # 按照服务器时间显示文件时间
                  autoindex_localtime on;
  
                  default_type application/octet-stream;
                  # 开启零复制。默认配置中，文件会先到nginx缓冲区，开启零复制后，文件跳过缓冲区，可以加快文件传输速度。
                  sendfile on;
                  # 限制零复制过程中每个连接的最大传输量
                  sendfile_max_chunk 1m;
                  # tcp_nopush与零复制配合使用，当数据包大于最大报文长度时才执行网络发送操作，从而提升网络利用率。
                  tcp_nopush on;
                  # 启用异步IO，需要配合direcio使用
                                          # aio on;
                  # 大于10MB的文件会采用直接IO的当时进行缓冲读取
                  directio 10m;
                  # 对齐文件系统块大小4096
                  directio_alignment 4096;
                  # 启用分块传输标识
                  chunked_transfer_encoding on;
                  # 文件输出的缓冲区大小为128KB
                  output_buffers 4 32k;
          }
  
          location / {
                  root   html;
                  index  index.html index.htm;
          }
  
  
          # redirect server error pages to the static page /50x.html
          #
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
                  root   html;
          }
  
  }
  ```

* 将`zip`文件放入`html`目录下，之后通过`http://IP地址:8788/docsify.zip`即可下载扩展名为`zip`文件