---
title: "[译]如何检测一个Hugo站点是开发环境还是生产环境"
date: 2022-06-19T12:09:19+08:00
lastmod: 2022-06-19T12:09:19+08:00
draft: false
keywords: ["hugo","go","development","production"]
description: "翻译外网中关于如何检测一个Hugo站点是开发环境还是生产环境的博文"
tags: ["hugo","go"]
categories: ["翻译","个人博客"]
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

本文翻译自[**How to check if a Hugo site is in development or production**](https://dev.to/tylerlwsmith/check-if-a-hugo-site-is-in-development-or-production-2beb)

<!--more-->

有时我们希望能在使用`Hugo`的时候基于开发环境或生产环境展示不同的内容，在之前我使用过`$.Site.IsServer`来检查是否运行于开发服务器，不过其无法实现在开发环境查看生产环境的显示样式。

## 检查环境

实际上`Hugo`有一系列的方式来区分生产环境和开发环境，不过[Hugo文档](https://gohugo.io/functions/hugo/)中关于如何区分它们的说明不太友好。



下述代码展示了在`Hugo`中检测环境的两种方式：

```go
{{ hugo.Environment }} returns "development" or "production"
{{ hugo.IsProduction }} returns true or false
```

当使用`hugo server`在本地开发时，默认被设置为`development`，当使用`hugo`来构建站点时，默认被设置为`production`。

## 基于环境动态展示

若想实现根据环境动态展示，可使用下述代码中的几种方法之一：

```go
{{ if eq hugo.Environment "development" }}
  I render when in development
{{ end }}

{{ if eq hugo.Environment "production" }}
  I render when in production
{{ end }}

{{ if hugo.IsProduction }}
  I render when in production
{{ end }}

{{ if not hugo.IsProduction }}
  I render when <strong>not</strong> in production, which means
  I would render if the environment were manually set to "test"
{{ end }}
```

## 手工设置环境变量

若要在编写博文时将环境设置为`production`，可使用下述命令启动`Hugo`服务器：

```bash
# 短命令
hugo server -e production

# 长命令
hugo server --environment production
```

也可通过设置系统环境变量来实现：

```bash
HUGO_ENVIRONMENT=production hugo server
```

`Hugo`要求能改变其配置的环境变量必须以`HUGO_`前缀开头，可在[Hugo环境变量说明](https://gohugo.io/getting-started/configuration/#configure-with-environment-variables)中了解具体信息。



如果我们想在构建时将环境变量设置为除了`production`之外的其它值，可使用下述几种方法之一，在这些方法中，我们将环境变量设置为`development`，也可设置为我们期望的其它任何值。

```bash
hugo -e development
hugo --environment development
HUGO_ENVIRONMENT=development hugo
```

希望此文能帮助你来构建自己的`Hugo`站点，喜欢此文章的话可以的点赞或评论！