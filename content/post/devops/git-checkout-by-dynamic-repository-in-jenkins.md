---
title: "在Jenkins中根据配置从不同的仓库中Checkout代码"
date: 2023-03-10T22:48:50+08:00
lastmod: 2023-03-10T22:48:50+08:00
draft: true
keywords: []
description: ""
tags: ["kubesphere","jenkins","nacos"]
categories: ["持续集成"]
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

mermaidDiagrams: 
  enable: true
  options: "{
     'theme':'forest',
     'themeVariables': {
        'noteTextColor':'red',
        'lineColor': '#0a908e',
        'fontSize':'14px'
     }
  }"
---

简要介绍

<!--more-->

展示效果

```mermaid
flowchart TD
    A(开始构建) -->B(选择对应参数)
    B -->|用于获取端口等配置| C[打包环境]
    B --> D[选择工程]
    B -->|对应工程Gitlab分支| E[选择分支]
    D --> F[Checkout代码]
    E --> F
    F --> |更新yaml等配置文件| G[更新配置]
    C --> G
    G --> H[代码编译]
    H --> I[(导出jar文件)]
    H --> J[镜像构建]
    J --> K[(导出镜像)]
```



