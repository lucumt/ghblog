---
title: "KubeSphere中整合LDAP后不生效的原因分析"
date: 2024-07-03T19:37:01+08:00
lastmod: 2024-07-03T19:37:01+08:00
draft: false
keywords: ["kubesphere","ldap"]
description: "KubeSphere中整合LDAP后不生效的原因分析"
tags: ["kubesphere","ldap"]
categories: ["工具使用","持续集成"]
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
borderImage: false

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

在给公司内部搭建的`KubeSphere`整合`LDAP`时，发现一开始整合后并不能立即使用`LDAP`登录，一直提示无效的用户名或密码，经过多次折腾后找到一些规律，简单记录下。

<!--more-->

仿照[**Kubesphere集成LDAP踩坑记录**](/post/devops/setting-ldap-for-kubesphere)一文配置好`LDAP`后，使用对应的账号去登录时，发现类似如下的报错

![LDAP账户登录失败](/blog_img/devops/ldap-integration-not-working/kubesphere-ldap-not-working.png "LDAP账户登录失败")  

可能的问题原因：

a) 确保`yaml`文件格式正常，可采用[YAML Formatter](https://jsonformatter.org/yaml-formatter)将`ks-install`配置文件格式化后与官网的进行对比

b) 配置完毕后需要重启相关pod，个人经常忘记这个步骤导致出错，来源参见[kubesphere 3.3.0配置LDAP之后不生效](https://ask.kubesphere.io/forum/d/8891-kubesphere-330ldap)

```bash
# 按顺序依次执行下述指令
kubectl -n kubesphere-system rollout restart deploy/ks-installer
kubectl -n kubesphere-system rollout restart deploy/ks-apiserver
```

c) 若`KubeSphere`的版本为`3.4.0`或以上，由于登录界面发生变化，需要显示选择`通过Ldap登录`[^1](个人觉得这种设计简直是多此一举)，默认是普通账户登录，此时也会给人造成错觉，以为登陆失败。

![LDAP登录选择](/blog_img/devops/ldap-integration-not-working/kubesphere-ldap-login.png "LDAP登录选择")  

[^1]: 参见 https://ask.kubesphere.io/forum/d/8260-330-ldap/24