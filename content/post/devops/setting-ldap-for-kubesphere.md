---
title: "Kubesphere集成LDAP踩坑记录"
date: 2022-09-04T09:35:50+08:00
lastmod: 2022-09-04T09S:35:50+08:00
draft: false
keywords: ["kubesphere","Jenkins","持续集成","持续部署","LDAP"]
description: "简要记录自己Kubesphere集成LDAP中所遇到的坑"
tags: ["kubesphere","jenkins","LDAP"]
categories: ["持续集成","系统集成"]
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

之前在公司内部推广[KubeSphere](https://kubesphere.io/)用于持续集成和部署，取得了不错的反馈，考虑到大规模使用的便利性以及之前已有[LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)整合其它系统的成熟经验，很自然的想将`LDAP`集成到`KubeSphere`中。原以为会很容易，一番折腾下来费了不好功夫(**`KubeSphere`要求使用`LDAP`时必须设置管理员账号和密码**)，简单记录下。

<!--more-->

公司内部的`LDAP`系统支持匿名登录，在给其它系统(如[yapi](https://github.com/fjc0k/docker-YApi))集成`LDAP`时，只需要输入对应的`host`、`uid`、`searchDN`即可，整合起来十分便捷，类似如下：

```json
"ldapLogin":{
   "enable":"true",
   "server":"ldap://192.168.0.2:389",
   "searchDn":"dc=lucumt,dc=info",
   "searchStandard":"uid",
   "usernameKey":"uid"
}
```

公司内部的`KubeSphere`版本为`v3.1.1`，在集成`LDAP`之前先去官网查看了相关说明，在[LDAP身份提供者](https://kubesphere.io/zh/docs/v3.3/access-control-and-account-management/external-authentication/use-an-ldap-service/)中有如下说明：

```yaml
spec:
 authentication:
   jwtSecret: ''
   maximumClockSkew: 10s
   multipleLogin: true
   oauthOptions:
     accessTokenMaxAge: 1h
     accessTokenInactivityTimeout: 30m
     identityProviders:
     - name: LDAP
       type: LDAPIdentityProvider
       mappingMethod: auto
       provider:
         host: 192.168.0.2:389
         managerDN: uid=root,cn=users,dc=nas
         managerPassword: ********
         userSearchBase: cn=users,dc=nas
         loginAttribute: uid
         mailAttribute: mail
```

可以看出其默认多了`managerDN`和`managerPassword`这两个管理员属性，但由于公司的`LDAP`默认对外不提供管理员账号和密码，只能匿名登录认证，且自己之前在其它系统中匿名配置`LDAP`都很顺畅，于是通过匿名方式配置如下：

```yaml
provider:
    host: 192.168.1.22:389
    #managerDN: uid=root,cn=users,dc=nas
    #managerPassword: ********
    userSearchBase: dc=lucumt,dc=info
    loginAttribute: uid
    mailAttribute: mail
```

将官方文档中要求的`managerDN`和`managerPassword`屏蔽掉，接着重启`KubeSphere`系统，登录过程出现如下提示：

![集成LDAP登录失败](/blog_img/devops/setting-ldap-for-kubesphere/kubesphere-ldap-login-failed.png "集成LDAP登录失败")  

错误信息提示说密码不能为空，但是我们登录的时候肯定有输入密码的，那只可能是`managerPassword`为空导致的。

查找`KubeSphere`官网相关说明发现其对于`managerDN`和`managerPassword`没有说明是选填项，结合前面的报错信息，可得出如下结论

> **`KubeSphere`不支持`LDAP`匿名登录**

![集成LDAP登录参数说明](/blog_img/devops/setting-ldap-for-kubesphere/kubesphere-ldap-setting-explanation.png "集成LDAP参数说明")  

于是只能找公司相关部门申请`LDAP`具有只读账号的管理员权限，配置类似如下，之后重启`KubeSphere`可正常集成`LDAP`登录。

```yaml
provider:
    host: 192.168.1.22:389
    managerDN: admin=cn,dc=lucumt,dc=info
    managerPassword: ********
    userSearchBase: dc=lucumt,dc=info
    loginAttribute: uid
    mailAttribute: mail
```

只能说`KubeSphere`在这方面做的不太好，已经在其官网反馈此问题!