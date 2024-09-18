---
title: "利用Grafana和Prometheus对系统进行监控"
date: 2022-12-15T10:56:54+08:00
lastmod: 2022-12-15T10:56:54+08:00
draft: false
keywords: ["grafana","promethus","系统监控","docker"]
description: "简要介绍如何基于docker环境利用Grafana和Prometheus对系统进行监控"
tags: ["devops","grafana","docker"]
categories: ["工具使用","容器化"]
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

简要介绍如何在`docker`环境下利用[**Grafana**](https://grafana.com/)和[**Prometheus**](https://prometheus.io/)对系统进行监控。

<!--more-->

## 软件安装

在[**给Grafana软件集成LDAP实现单点登录**](/post/ldap/add-ldap-support-for-grafana/)一文中简要说明了如何基于`docker`安装`Grafana`，本节出于简化使用与维护的目的，将`Grafana`与`Prometheus`合并到一个`docker-compose.yml`文件中，相关的文件如下

`docker-compose.yml`文件

```yaml
version: "3"
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: "prometheus_custom"
    restart: always
    ports:
      - "9090:9090"
    volumes:
      - "$PWD/prometheus.yml:/etc/prometheus/prometheus.yml"
      - "$PWD/prometheus_data:/prometheus"
  grafana:
    image: grafana/grafana
    container_name: "grafana_custom"
    ports:
      - "3000:3000"
    restart: always
    volumes:
      - "$PWD/grafana_data:/var/lib/grafana"
      - "$PWD/ldap.toml:/etc/grafana/ldap.toml"
    environment:
      # 管理员账号
      - GF_SECURITY_ADMIN_USER=admin
      # 管理员密码
      - GF_SECURITY_ADMIN_PASSWORD=xxx
      - GF_AUTH_LDAP_ENABLED=true
```

`ldap.toml`文件，用于`Grafana`接入`LDAP`

```toml
[[servers]]
host = "10.10.0.55"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = false
bind_dn = "cn=xxx,dc=chinahirain,dc=com"
bind_password = 'xxx'
search_filter = "(uid=%s)"
search_base_dns = ["dc=chinahirain,dc=com"]

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

`prometheus.yml`文件用于配置要接入`Prometheus`的系统

```yaml
global:
  scrape_interval:     15s # 默认抓取周期
  external_labels:
    monitor: 'codelab-monitor'
scrape_configs:
  - job_name: 'node-exporter' #服务的名称
    scrape_interval: 5s
    metrics_path: /metrics  #获取指标的url
    static_configs:
      # 这个为监听指定服务服务的ip和port
      - targets: ['10.30.31.56:9188','10.30.5.113:9188']
```

上述这些文件必须位于同一目录下，之后通过`docker-compose`启动

```bash
root@lucumt:~/grafana# ls
docker-compose.yml  grafana_data  ldap.toml  prometheus.yml  prometheus_data
root@lucumt:~/grafana# docker-compose up -d
```

## 系统接入

主要基于`node exporter`实现，参考 [**centos7安装node export**](https://www.cnblogs.com/rainbow-tan/p/16623772.html) 一文进行相关操作。

1. 在安装有`node exporter`服务器上的`/usr/local/bin/`目录下复制名为`node_exporter`的文件到目标机器的`/usr/local/bin/`目录下，并将其赋予可执行权限

   ```bash
   chmod +x /usr/local/bin/node_exporter
   ```

2. 执行`vi /etc/systemd/system/node_exporter.service`并输入下述内容

   ```toml
   [Unit]
   Description=cicd_exporter
   After=network.target
   
   [Service]
   ExecStart=/usr/local/bin/node_exporter --web.listen-address=:9188
   Restart=on-failure
   
   [Install]
   WantedBy=multi-user.target
   ```
   
3. 执行下述命令将其设置为`Linux`服务同时开机自启动

   ```bash
   systemctl daemon-reload && systemctl start node_exporter && systemctl enable node_exporter
   ```

4. 在目标机器的浏览器中输入`http://127.0.0.1:9188`若能正常显示，则表示`node_exporter`安装成功

5. 在安装`Grafana`的服务器上找到前述的`prometheus.yml`，在其`targets`下面添加对应的`node_exporter`服务，然后通过`docker-compose restart`重启相关服务

6. 在`Grafana`中查看新添加的服务器是否集成成功