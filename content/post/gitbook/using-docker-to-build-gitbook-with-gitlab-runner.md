---
title: "在Docker中构建GitBook并整合GitLab Runner的使用经验分享"
date: 2023-10-07T10:14:02+08:00
lastmod: 2023-10-07T10:14:02+08:00
draft: true
keywords: ["docker","gitbook"]
description: "简要介绍如何通过Docker创建GitBook并通过GitLab Runner实现GitBook文档内容的自动更新"
tags: ["gitbook","docker","gitlab-runner"]
categories: ["工具使用","容器化"]
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
  enable: true
  options: "{
              'x': 0,
              'y': 10,
              'width':1,
              'line-width': 1.3,
              'line-length': 50,
              'text-margin': 10,
              'font-size': 14,
              'font-color': 'black',
              'line-color': 'black',
              'element-color': 'black',
              'fill': 'white',
              'yes-text': 'yes',
              'no-text': 'no',
              'arrow-end': 'block',
              'font-weight': 'bold',
              'font-color': 'white',
              'scale': 1,
              'symbols': {
              },
              'flowstate': {
                'approved': {
					'fill': '#58C4A3',
					'font-size': 12
				},
				'current': {
					'fill': '#008080',
					'font-color': 'white',
					'font-weight': 'bold'
				},
				'future': {
					'fill': '#8A624A'
				},
				'success': {
					'fill': '#0B6623',
					'font-color': 'white',
					'font-weight': 'bold'
				},
				'invalid': {
					'fill': '#6c5696'
				},
				'past': {
					'fill': '#CCCCCC',
					'font-size': 12
				},
				'select': {
					'fill': '#E1AD01',
					'font-size': 12
				},
				'rejected': {
					'fill': '#C45879',
					'font-size': 12
				},
				'request': {
					'fill': '#569656'
				},
				'yellowgreen':{
					'fill': 'yellowgreen'
				},
				'failed':{
					'fill': '#800020'
				},
                'approved': {'fill': 'peru'}
              }
            }"

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

由于`Docsify`采用单页渲染的方式，在初次打开是特别慢，近期将团队内部的知识库工具从`Docsify`迁移到基于`Docker`搭建的`GitBook`，在这其中踩了一些坑，简单记录下。

<!--more-->

# 背景

## 技术选型

由于`Docsify`采用[**SPA**](https://en.wikipedia.org/wiki/Single-page_application)模式开发，不会生成静态页面，故随着内容与页面的增多，其初次加载时会花费较多时间进行全局渲染，导致显示很慢，浪费时间等于谋财害命，需要将其替换为其它框架！

由于知识库的使用群体除了软件开发人员，还有产品设计师、测试工程师等不具备很强代码开发能力的群体，故在替换之前对要采用的新框架确定了如下目标：

* 采用静态页面生成，避免每次浏览时都需要耗时渲染
* 采用纯`Markdown`语法，避免额外的学习与使用成本
* 界面美观，适合做文档展示，无须做过多配置

基于网络上的资料，初步选定`Docsify`、`GitBook`、`Hexo`和`Hugo`这4种框架，基于实际使用需求对它们进行对比如下：

|             | 开发语言  | 构建速度 | 展示方式   | 格式要求               | 文档 / 社区 | 目录生成                   | 优点                         | 缺点                              |
| ----------- | --------- | -------- | ---------- | ---------------------- | ----------- | -------------------------- | ---------------------------- | --------------------------------- |
| **Docsify** | `Vue`     | 一般     | 单页面渲染 | 无                     | 一般        | 可通过插件自动生成         | 环境安装简单                 | 初次打开加载很慢                  |
| **GitBook** | `Node.js` | 一般     | 静态页面   | 无                     | 一般        | 需自己在`SUMMARY.md`中生成 | 样式美观，专门用于做文档管理 | 官方专注于付费版，非付费版bug较多 |
| **Hexo**    | `Node.js` | 一般     | 静态页面   | 需添加日期、标题等信息 | 丰富        | 依赖于所选的主题           | 社区活跃，使用度很高         | 编译速度慢，主要适合做博客        |
| **Hugo**    | `Golang`  | 快       | 静态页面   | 需添加日期、标题等信息 | 丰富        | 依赖于所选的主题           | 构建速度快，使用度很高       | 文档不完善，主要适合做博客        |

基于上述对比，可发现虽然`Hugo`的构建速度最快，但适合做博客，对于知识库这种文档系统不是特别合适，而`GitBook`虽然构建速度没有`Hugo`快，但`GitBook`更侧重于浏览，其构建只会在文档内容有更新时才发生，频率相对较小，同时`GitBook`从名字上即可看出是专门用于知识库文档管理的，综合考虑下决定采用`GitBook`来替换`Docsify`。

## 使用流程

由于文档同代码一样都是采用`GitLab`管理的，为了实现在文档更新时能够自动化的部署，准备采用`GitLab Runner`+`GitBook`+`Nginx`的方案，整体流程图如下：

```flow
start=>start: 开始
end=>start: 更新完毕
write_md=>operation: 编写Markdown文档 | current
commit_md=>operation: 提交文档到GitLab | success
trigger_build=>operation: GitLab Runner触发构建 | rejected
clone_code=>subroutine: GitLab Runner拉取代码 | approved
build_page=>subroutine: GitBook构建页面 | approved
deploy_page=>subroutine: GitLab Runner部署页面到Nginx | success

start->write_md(right)->commit_md->trigger_build
trigger_build->clone_code(right)->build_page->deploy_page->end
```

同时考虑到降低对部署环境的侵入性，将`GitLab Runner`和`Nginx`采用`Docker`进行容器化部署。

# 环境搭建

## 镜像构建



## GitLab Runner注册

## 功能验证

# 插件管理

## 常用插件

## 代码高亮

## 图表插件

已有的插件太老旧，自己fork代码后对它们进行了更新，项目位于[**gitbook-plugin-fox**](https://github.com/gitbook-plugin-fox)，目前包含如下插件：

* [**gitbook-plugin-mermaid-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-mermaid-fox)，`GitBook`中支持新版的[**flowchart.js**](https://flowchart.js.org/)图表，展示效果如下

  ![GitBook中展示Flowchart图表](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-flowchart-chart-display.png "GitBook中展示Flowchart图表")

* [**gitbook-plugin-mermaid-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-mermaid-fox)，`GitBook`中支持新版的[**Mermaid**](https://mermaid.js.org/)图表，展示效果如下

  ![GitBook中展示Mermaid图表](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-mermaid-chart-display.png "GitBook中展示Mermaid图表")

# 其它设置

## 中文汉化

## 自定义样式

