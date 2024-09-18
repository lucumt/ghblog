---
title: "[非原创]解决Gitbook安装过程中出现的Error: Missing required argument #1错误"
date: 2023-10-13T19:10:56+08:00
lastmod: 2023-10-3T19:10:56+08:00
draft: false
keywords: ["gitbook install","gitbook plugin","npm"]
description: "基于CSDN博文，记录解决Gitbook安装过程中出现的Error: Missing required argument #1"
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

本文来源于个人在使用`GitBook`中遇到的问题，尽管自己通过 **[gitbook出现if (args[ii] == null) throw missingRequiredArg(ii 解决](https://blog.csdn.net/AntiO2/article/details/125964797)** 这篇文章很快的找到了问题的解决方案，但该文章发布在CSDN，而CSDN的使用体验糟糕已是尽人皆知(此处不得不说一声 **CSDN fuck you!**)。

为了给其他使用者提供一个简洁干净的参考，故写此文章。

<!--more-->

## 问题场景

通过`gitbook install` 进行相关插件的安装。

## 问题描述

通过`gitbook install` 进行相关插件的安装，终端出现类似如下错误导致安装过程失败

```powershell
D:\code\gitbook-doc>gitbook install
info: installing 24 plugins using npm@3.9.2
info:
info: installing plugin "mermaid-fox"
info: install plugin "mermaid-fox" (*) from NPM with version 0.0.3
fetchMetadata -> network  \ |##################-----------------------------------------------------------------------|
C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\node_modules\aproba\index.js:25
    if (args[ii] == null) throw missingRequiredArg(ii)
                          ^

Error: Missing required argument #1
    at andLogAndFinish (C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\lib\fetch-package-metadata.js:31:3)
    at fetchPackageMetadata (C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\lib\fetch-package-metadata.js:51:22)
    at resolveWithNewModule (C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\lib\install\deps.js:490:12)
    at C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\lib\install\deps.js:491:7
    at C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\node_modules\iferr\index.js:13:50
    at C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\lib\fetch-package-metadata.js:37:12
    at addRequestedAndFinish (C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\lib\fetch-package-metadata.js:67:5)
    at returnAndAddMetadata (C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\lib\fetch-package-metadata.js:121:7)
    at pickVersionFromRegistryDocument (C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\lib\fetch-package-metadata.js:138:20)
    at C:\Users\yunqiang.lu\.gitbook\versions\3.2.3\node_modules\npm\node_modules\iferr\index.js:13:50 {
  code: 'EMISSINGARG'
}
```

## 原因分析

原作者不知道，本人也不知道，期待有大神能给出具体原因分析。

## 解决方案

需要用`npm`先手工安装对应的插件，之后再执行`gitbook install`就不会出错。

以本文为例，出错的插件为`gitbook-plugin-mermaid-fox`[^1]，则按照下述方式执行即可

```bash
# 必须先执行该指令
npm install gitbook-plugin-mermaid-fox

gitbook install
```

[^1]: `GitBook`中的插件要求必须以`gitbook-plugin-`开头

