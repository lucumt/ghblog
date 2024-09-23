---
title: "给Gitlab添加LDAP登录认证"
date: 2023-04-14T11:22:34+08:00
lastmod: 2023-04-14T11:22:34+08:00
draft: false
keywords: ["ldap","docker","gitlab"]
description: "给Gitlab添加LDAP登录认证并禁止普通账号登录，主要是给自建LDAP提供使用改进参考"
tags: ["ldap","docker"]
categories: ["工具使用","系统集成"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
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

简要记录如何基于`docker`创建`Gitlab`并集成`ldap`[^1]且仅允许`LDAP`账号登录。

<!--more-->

## 安装步骤

1. 创建一个名为`gitlab_test`的文件夹，之后创建一个名为`docker-compose.yml`的文件，写入如下内容

   ```yaml
   version: "3"
   services:
     nacos:
       image: gitlab/gitlab-ce
       restart: always
       container_name: gitlab_custom
       ports:
         - 3555:80
       environment:
         - TZ=Asia/Shanghai
       volumes:
         - $PWD/etc:/etc/gitlab
         - $PWD/log:/var/log/gitlab
         - $PWD/opt:/var/opt/gitlab
   ```

2. 运行`docker-compose up -d`命令，结合服务器性能以及网络环境等因素，`gitlab`需要2-3分钟才能完成启动成功，之后在当前目录下会自动创建相关的挂载目录同时写入相关映射文件

   ![gitlab自动挂载数据](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-volume-folder-and-data.png "gitlab自动挂载数据") 

   输入`http://IP地址:3555`可正常显示登录页面

   ![gitlab初始登录页面](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-init-login-page.png "gitlab初始登录页面") 

3. 在终端运行`docker exec -it gitlab_custom cat  /etc/gitlab/initial_root_password`会显示`root`账号的初始密码，可利用改密码登录系统然后修改成更易于记忆的密码

   ![root账号初始密码](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-cat-init-password.png "root账号初始密码") 

4. 在终端运行`vim etc/gitlab.rb`添加如下图所示的`LDAP`相关配置

   ![修改配置文件集成ldap](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-enable-ldap-config.png "修改配置文件集成ldap") 

5. 在终端运行`docker exec -it gitlab_custom /bin/bash gitlab-ctl reconfigure`来更新`gitlab`配置

   ![gitlab更新配置](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-ctl-reconfigure.png "gitlab更新配置") 

6. 在终端运行`docker exec -it gitlab_custom /bin/bash gitlab-rake gitlab:ldap:check`，若出现类似如下结果则表示`LDAP`配置正常，可用单点账号登录

   ![ldap检查结果正常](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-retakke-ldap-check-result.png "ldap检查结果正常") 

   重新登录`gitlab`时可发现其已经支持`LDAP`，至此集成`LDAP`的工作初步完成。

   ![gitlab登录支持ldap](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-login-with-ldap-page.png "gitlab登录支持ldap") 

## 添加额外的管理员

`gitlab`默认只有`root`这1个admin账号，当禁止非`LDAP`方式登录时会将`root`账号的登录一并禁止掉，故需要预先将一些`LDAP`账户设置为管理员账号，操作过程如下：

1. 在终端运行`docker exec -it gitlab_custom /bin/bash gitlab-rails console`进入`ruby`的交互式界面

   ![进入gitlab的ruby交互界面](/blog_img/ldap/add-ldap-support-for-gitlab/enter-gitlab-ruby-console-environment.png "进入gitlab的ruby交互界面") 

2. 在控制台中依次执行如下命令

   ```ruby
   user = User.find_by(username: 'yunqiang.lu')
   user.admin = true
   user.save!
   exit
   ```

   ![通过gitlab的ruby控制台设置管理员](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-ruby-console-add-admin.png "通过gitlab的ruby控制台设置管理员") 

3. 在终端重新执行`docker exec -it gitlab_custom /bin/bash gitlab-ctl reconfigure`

   ![gitlab刷新ldap管理员配置](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-reconfigure-ldap-admin-result.png "gitlab刷新ldap管理员配置") 

4. 采用前述过程中配置的账号登录`gitlab`，若在左上角的`Menu`菜单下出现`Admin`子菜单则表示管理员账号添加成功!

   ![gitlab添加管理员操作成功](/blog_img/ldap/add-ldap-support-for-gitlab/login-gitlab-and-show-admin-menu.png "gitlab添加管理员操作成功") 

## 只允许LDAP登录

主要参考[How to disable standard authentication](https://github.com/sameersbn/docker-gitlab/issues/2604)进行操作：

1. 以管理员账号登录`gitlab`，依次点击`Admin`->`Settings`->`General`

2. 在`Sign-up restrictions`下取消勾选`sign-up enabled`

   ![gitlab取消sign up](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-disable-sign-up.png "gitlab取消sign up") 

3. 在`Sign in restrictions`下取消勾选`Sign in restrictions`

   ![gitlab取消web authentication](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-disable-web-authentication.png "gitlab取消web authentication") 

4. 测试之后重新打开`gitlab`登录页面，可发现其支持`LDAP`登录，至此全部过程操作完成!

   ![gitlab只支持ldap登录](/blog_img/ldap/add-ldap-support-for-gitlab/gitlab-only-support-ldap-login.png "gitlab只支持ldap登录") 

[^1]: 采用的是`gitlab-ce`而非`gitlab-ee`