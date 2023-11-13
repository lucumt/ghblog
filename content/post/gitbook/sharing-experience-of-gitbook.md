---
title: "GitBook使用经验分享"
date: 2023-10-07T10:14:02+08:00
lastmod: 2023-10-07T10:14:02+08:00
draft: true
keywords: []
description: ""
tags: ["gitbook"]
categories: ["工具使用"]
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

```flow
start_login=>start: 开始登录
login_success=>end: 登录成功
login_failed=>end: 登录失败
user_ldap_check=>operation: 检查用户类型 | request
user_ldap_check_result=>condition: 是否为LDAP用户 | approved

local_user_auth=>operation: 本地账户验证 | current
local_user_auth_result=>condition: 本地验证结果 | rejected

ldap_user_auth=>operation: LDAP账户验证 | current
ldap_user_auth_result=>condition: LDAP验证结果 | rejected
create_ldap_user=>subroutine: 创建LDAP用户 | request

start_login(right)->user_ldap_check(right)->user_ldap_check_result
user_ldap_check_result(no)->local_user_auth->local_user_auth_result
user_ldap_check_result(yes)->ldap_user_auth->ldap_user_auth_result

local_user_auth_result(no)->login_failed
local_user_auth_result(yes)->login_success

ldap_user_auth->ldap_user_auth_result
```

