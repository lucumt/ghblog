---
title: "在Docker中构建GitBook并整合GitLab Runner的使用经验分享"
date: 2023-10-07T10:14:02+08:00
lastmod: 2023-10-07T10:14:02+08:00
draft: false
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
              'yes-text': 'yes',
              'no-text': 'no',
              'line-width': 1.3,
              'line-length': 30,
              'text-margin': 10,
              'font-size': 15,
              'font-color': 'black',
              'line-color': 'black',
              'element-color': 'black',
              'fill': 'white',
              'arrow-end': 'block',
              'font-weight': 'bold',
              'font-color': 'white',
              'scale': 0.8,
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
					'font-size': 12,
                    'yes-text': '是',
                    'no-text': '否'
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

由于`Docsify`采用单页渲染的方式，在初次打开是特别慢，近期将团队内部的知识库工具从`Docsify`迁移到基于`Docker`搭建的`GitBook`并结合`GitLab Runner`实现自动更新，在这其中踩了一些坑，简单记录下。

<!--more-->

## 背景

### 技术选型

由于`Docsify`采用[**SPA**](https://en.wikipedia.org/wiki/Single-page_application)模式开发，不会生成静态`HTML`页面，故随着内容与页面的增多，其初次加载时会花费较多时间进行全局渲染，导致显示很慢，浪费时间等于谋财害命，需要将其替换为其它框架！

由于知识库的使用群体除了软件开发人员，还有产品设计师、测试工程师等不具备很强代码开发能力的群体，故在替换之前对要采用的新框架确定了如下目标：

* 采用静态页面生成，避免每次浏览时都需要耗时渲染
* 采用纯`Markdown`语法，避免额外的学习与使用成本
* 界面美观，适合做文档展示，无须做过多配置
* 插件生态丰富，无须自己额外开发太多的插件

基于网络上的资料，初步选定`Docsify`、`GitBook`、`Hexo`和`Hugo`这4种框架，基于实际使用需求对它们进行对比如下：

|             | 开发语言  | 构建速度 | 展示方式   | 格式要求               | 文档 / 社区 | 目录生成                   | 优点                         | 缺点                              |
| ----------- | --------- | -------- | ---------- | ---------------------- | ----------- | -------------------------- | ---------------------------- | --------------------------------- |
| **Docsify** | `Vue`     | 一般     | 单页面渲染 | 无                     | 一般        | 可通过插件自动生成         | 环境安装简单                 | 初次打开加载很慢                  |
| **GitBook** | `Node.js` | 一般     | 静态页面   | 无                     | 一般        | 需自己在`SUMMARY.md`中生成 | 样式美观，专门用于做文档管理 | 官方专注于付费版，非付费版bug较多 |
| **Hexo**    | `Node.js` | 一般     | 静态页面   | 需添加日期、标题等信息 | 丰富        | 依赖于所选的主题           | 社区活跃，使用度很高         | 编译速度慢，主要适合做博客        |
| **Hugo**    | `Golang`  | 快       | 静态页面   | 需添加日期、标题等信息 | 丰富        | 依赖于所选的主题           | 构建速度快，使用度很高       | 文档不完善，主要适合做博客        |

基于上述对比，可发现虽然`Hugo`的构建速度最快，但适合做博客，对于知识库这种文档系统不是特别合适，而`GitBook`虽然构建速度没有`Hugo`快，但`GitBook`更侧重于浏览，其构建只会在文档内容有更新时才发生，频率相对较小，同时`GitBook`从名字上即可看出是专门用于知识库文档管理的，综合考虑下决定采用`GitBook`来替换`Docsify`。

同时为了降低`GitBook`的侵入性，指定如下规则：

1. `GitLab Runner`和`Nginx`采用`Docker`进行容器化部署，避免对部署环境的强依赖
2. 通过`Docker`在`GitLab Runner`中安装`GitBook`环境，使得实际的文档项目与`GitBook`解耦

### 使用流程

由于文档同代码一样都是采用`GitLab`管理的，为了实现在文档更新时能够自动化的部署，准备采用`GitLab Runner`+`GitBook`+`Nginx`的方案，整体流程图如下：

![GitBook结合GitLab Runner使用流程](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-gitlab-usage-flow.png "GitBook结合GitLab Runner使用流程")

在实际使用时，相关用户只需要在按照常规的流程在本地基于`Markdown`格式编写文档并提交到`GitLab`中，之后`GitLab Runner`会自动化的进行文档构建与文档生成，依赖于宿主物理机的配置和文档数量，整个过程通常不超过5秒钟，之后在浏览器中刷新即可查看生成的`HTML`静态文件。

## 环境搭建

### 相关说明

出于简化使用流程与加快构建速度的考虑，将`GitLab Runner`与`GitBook`都放置到同一个容器中。在`GitLab Runner`的`Docker`容器中安装完毕`GitBook`环境后，还需将`GitLab Runner`注册到对应的`GitLab`仓库中，可通过`Shell`脚本将此过程固化下来，整个环境搭建流程如下：

![GitBook安装流程](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-install-flow.png "GitBook安装")

**说明**：由于`GitBook`是基于`Nodejs`开发的，而`Nodejs`和`NPM`对版本有强相关的依赖，版本过高或过低都会导致`GitBook`以及相关的插件无法安装，故本次采用了如下**较旧的版本**(采用`Docker`容器化部署不会造成额外的负面影响 )

|                   | 版本                           | 备注                                                         |
| ----------------- | ------------------------------ | ------------------------------------------------------------ |
| **GitLab Runner** | `gitlab/gitlab-runner:v15.5.2` |                                                              |
| **GitBook Cli**   | `gitbook-cli@2.1.2`            | 若版本不对，会导致依赖问题，请参见[**Gitbook-cli install error TypeError: cb.apply is not a function inside graceful-fs**](https://stackoverflow.com/questions/64211386/gitbook-cli-install-error-typeerror-cb-apply-is-not-a-function-inside-graceful) |
| **GitBook**       | `3.2.3`                        |                                                              |
| **Nodejs**        | `v12.22.12`                    |                                                              |
| **NPM**           | `7.5.2`                        |                                                              |

### 操作流程

下述步骤展示了从头开始构建`GitLab Runner`与`GitBook`的过程

1. 在宿主机上执行如下指令安装[**jq**](https://github.com/jqlang/jq)指令，后续会用其来解析`GitLab`中返回的`JSON`数据

   ```bash
   # 在宿主机上安装jq，用于解析json文件
   apt install -y jq
   ```

2. 安装之前请仿照下图将本文最后的[**文件&脚本**](#文件脚本)

   ![GitBook镜像构建前准备](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-build-file-folder.png "GitBook镜像构建前准备")

3. 运行下述指令构建需要的镜像

   ```bash
   docker build -t gitbook_custom:v1.0
   ```

   `Dockerfile`在构建过程中会对它认为没有变化的文件和日志输出进行缓存，要想查看全部日志并取消缓存，可用如下指令

   ```bash
   docker build -t gitbook_custom:v1.0  --progress string . --no-cache
   ```

4. 执行下述指令启动`Docker`容器并执行相关的初始化操作

   ```bash
   docker-compose down && docker-compose up -d && bash gitlab_runner_config_init.sh
   ```

5. 安装过程中会有类似如下输出:

   ![GitBook安装过程1](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-install-output-1.png "GitBook安装过程1")

   若最后输出结果类似如下，则表示整个脚本安装过程顺利完成

   ![GitBook安装过程2](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-install-output-2.png "GitBook安装过程2")

6. 在对应`GitLab`项目中依次点击`Settings`->`CI/CD`->`Runners`，若出现类似如下界面则表示`GitLab Runner`正确安装与配置完毕，接下来即可验证`GitBook`是否能正常工作

   ![GitLab Runner注册结果](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitlab-runner-register-result.png "GitLab Runner注册结果")

### 测试验证

#### 自动构建

由于[**自动构建脚本**](#自动构建脚本)中有如下配置，其中构建阶段的值被设置为`build`会导致每次代码发生变更时都执行自动构建

```yaml
stages:
  - build

# build阶段执行的操作命令 
build:
  stage: build
```

当我们通过`git push`将代码推送到`GitLab`后，在对应的`GitLab`工程中依次点击`CI/CD`->`Pipelines`，会有类似如下输出，可以看到此时`GitLab Runner`正在执行构建

![GitLab Runner正在执行](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitlab-pipeline-running.png "GitLab Runner正在执行")

点击对应的按钮可查看构建执行日志，类似如下：

![GitLab Runner执行日志结果](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitlab-pipeline-running-log.png "GitLab Runner执行日志结果")

之后可通过`Nginx`对应端口访问相应的`GitBook`页面，可参考[**使用展示**](#使用展示)。

#### 手工构建

除了自动构建之外，也可点击对应的按钮进行手工提交，此种场景一般用于文档代码没有变更而对`GitBook`的样式或插件进行了修改，想提前验证结果，操作步骤如下：

在对应的`GitLab`工程中依次点击`CI/CD`->`Pipelines`，在出现的菜单中点击右上角的`Run pipeline`按钮

![GitLab Runner手工触发](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitlab-pipeline-run-button.png "GitLab Runner手工触发")

在出现的界面中再次点击`Run pipeline`按钮进行确认，触发手工执行，之后的步骤与自动构建时相同。

![GitLab Runner手工触发确认](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitlab-pipeline-run-confirm-button.png "GitLab Runner手工触发确认")

## 插件管理

`GitBook`在文档管理领域广受欢迎的一个很重要的原因是其丰富的插件生态，插件可快速集成，也可仿照别人的插件根据自己的需求快速开发新的插件，个人项目中用到的插件如下

### 常用插件

| 插件                                                         | 作用                                                         | 备注                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| [**gitbook-plugin-prism-codetab-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-prism-codetab-fox) | 对代码采用[**Prims.js**](https://prismjs.com/)进行高亮，同时可根据需求对代码进行分组 | 需要在`book.js`中通过 `-highlight`的方式屏蔽默认的高亮插件  |
| [**gitbook-plugin-mermaid-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-mermaid-fox) | 支持[**Mermaid**](https://mermaid.js.org/)图表功能           | 实际使用过程中发现部分图表无法展示                          |
| [**gitbook-plugin-flowchart-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-flowchart-fox) | 用于支持[**flowchart**](https://flowchart.js.org/)图表       | 实际使用过程中发现该插件bug较多，在组件过多时会出现样式混乱 |
| [**gitbook-plugin-advanced-emoji**](https://github.com/codeclou/gitbook-plugin-advanced-emoji) | 用于展示[**Emoji**](https://getemoji.com/)表情               |                                                             |
| [**gitbook-plugin-chart**](https://github.com/csbun/gitbook-plugin-chart#readme) | 利用[**C3.js**](http://c3js.org/)或[**Highcharts**](http://www.highcharts.com/)来展示图表 |                                                             |
| [**gitbook-plugin-popup**](https://github.com/somax/gitbook-plugin-popup) | 在新窗口中打开图片                                           |                                                             |
| [**gitbook-plugin-accordion**](https://github.com/artalar/gitbook-plugin-accordion/) | 手风琴插件，用于展开或折叠相关内容                           |                                                             |
| [**plugin-katex**](https://github.com/GitbookIO/plugin-katex) | 支持基于[**KaTeX**](https://katex.org/)的数学公式展示        |                                                             |
| [**gitbook-plugin-search-pro**](https://github.com/gitbook-plugins/gitbook-plugin-search-pro) | 支持中文搜索的插件                                           | 需要在`book.js`中通过 `-search`的方式屏蔽默认的搜索插件[^1] |
| [**gitbook-plugin-hide-element**](https://github.com/gonjay/gitbook-plugin-hide-element) | 隐藏特定元素                                                 |                                                             |
| [**gitbook-plugin-chapter-fold**](https://github.com/ColinCollins/gitbook-plugin-chapter-fold) | 页面上方的导航目录折叠                                       |                                                             |
| [**gitbook-plugin-code**](https://github.com/davidmogar/gitbook-plugin-code) | 给代码块添加行号与复制按钮                                   |                                                             |
| [**gitbook-plugin-splitter**](https://github.com/yoshidax/gitbook-plugin-splitter) | 页面左侧的侧边栏可动态调节                                   | 该插件与`xmind`插件一并使用时会导致后者无法正常使用         |
| [**gitbook-plugin-expandable-chapters-small**](https://github.com/cjdbarlow/gitbook-plugin-expandable-chapters-small) | 页面右侧的目录可展开或收缩                                   |                                                             |
| [**gitbook-plugin-tbfed-pagefooter**](https://github.com/FastGitORG/gitbook-plugin-tbfed-pagefooter) | 页面显示定制化的页脚信息                                     |                                                             |
| [**gitbook-plugin-ancre-navigation**](https://github.com/thomas88100/gitbook-plugin-ancre-navigation) | 页内导航回到顶部                                             |                                                             |
| [**gitbook-plugin-sharing**](https://github.com/GitbookIO/plugin-sharing) | 将特定页面分享到其它平台的按钮插件                           | 需要在`book.js`中通过 `-sharing`的方式屏蔽默认的分享插件    |
| [**gitbook-plugin-theme-comscore**](https://plugins.gitbook.com/plugin/theme-comscore) | `GitBook`彩色主题                                            |                                                             |
| [**gitbook-plugin-favicon**](https://github.com/menduo/gitbook-plugin-favicon) | 添加favicon图标                                              |                                                             |
| [**gitbook-plugin-flexible-alerts**](https://github.com/fzankl/gitbook-plugin-flexible-alerts) | 自定义的提示说明样式                                         |                                                             |

### 自定义插件

已有的插件太老旧，自己`fork`代码后对它们进行了更新，项目位于[**gitbook-plugin-fox**](https://github.com/gitbook-plugin-fox)，目前包含如下插件：

* [**gitbook-plugin-mermaid-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-mermaid-fox)，`GitBook`中支持新版的[**flowchart.js**](https://flowchart.js.org/)图表，展示效果如下

  ![GitBook中展示Flowchart图表](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-flowchart-chart-display.png "GitBook中展示Flowchart图表")

* [**gitbook-plugin-mermaid-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-mermaid-fox)，`GitBook`中支持新版的[**Mermaid**](https://mermaid.js.org/)图表，展示效果如下

  ![GitBook中展示Mermaid图表](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-mermaid-chart-display.png "GitBook中展示Mermaid图表")
  
* [**gitbook-plugin-prism-codetab-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-prism-codetab-fox)，将[**gitbook-plugin-prism**](https://github.com/gaearon/gitbook-plugin-prism)与[**gitbook-plugin-codetabs**](https://github.com/GitbookIO/plugin-codetabs)这两个插件整合为一个插件，在实现代码高亮的同时还能对代码进行分组，展示效果如下：

  ![GitBook中展示代码分组](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-codetab-fox-display.png "GitBook中展示代码分组")

## 其它设置

### 中文汉化

`GitBook`中的默认语言为英文，如下图所示，某些提示信息以英文展示，不方便使用

![GitBook默认英文展示](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-default-en-language.png "GitBook默认英文展示")

可通过在`book.js`中添加`language:'zh-hans'`来将默认语言修改为中文，修改后的显示效果如下：

![GitBook切换为中文显示](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-set-zh-language.png "GitBook切换为中文显示")

### 自定义目录

如下所示`GitBook`中默认生成的菜单为`Introduction`，不太直观，可根据实际需求动态修改(对于菜单目录的生成可参考[**自动生成目录**](#自动生成目录)实现)

![GitBook默认菜单展示](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-default-menu.png "GitBook默认菜单展示")

将默认目录从

```markdown
* [Introduction](README.md)
```

修改为

```markdown
* [概览](README.md)
```

之后的展示效果类似如下

![GitBook自定义菜单展示](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-custom-menu.png "GitBook自定义菜单展示")

### 自定义样式

若`GitBook`中的样式不满足我们的需求，可通过添加自定义`CSS`文件进行定制，实际使用中发现该文件必须叫做`website.css`且需位于生成的静态文件目录下才生效。

```json
"styles": {
    "website": "./styles/website.css",
}
```

## 使用展示

可通过`Nginx`对应端口访问相应的`GitBook`页面，个人项目中的界面类似如下，供参考

![GitBook页面浏览](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-web-page-view.png "GitBook页面浏览")

## 文件&脚本

### Dockerfile脚本

用于构建同时包含`GitLab Runner`与`GitBook`的容器

```dockerfile
FROM gitlab/gitlab-runner:v15.5.2

# 更改为公司内部的镜像源
RUN rm -rf /etc/apt/sources.list && touch /etc/apt/sources.list

RUN echo 'deb [trusted=yes] https://mirrors.xxx.com/repository/Debian/ bullseye main non-free contrib' >> /etc/apt/sources.list
RUN echo 'deb-src [trusted=yes] https://mirrors.xxx.com/repository/Debian/ bullseye main non-free contrib' >> /etc/apt/sources.list
RUN echo 'deb [trusted=yes] https://mirrors.xxx.com/repository/Debian/ bullseye-updates main non-free contrib' >> /etc/apt/sources.list
RUN echo 'deb-src [trusted=yes] https://mirrors.xxx.com/repository/Debian/ bullseye-updates main non-free contrib' >> /etc/apt/sources.list
RUN echo 'deb [trusted=yes] https://mirrors.xxx.com/repository/Debian/ bullseye-backports main non-free contrib' >> /etc/apt/sources.list
RUN echo 'deb-src [trusted=yes] https://mirrors.xxx.com/repository/Debian/ bullseye-backports main non-free contrib' >> /etc/apt/sources.list

# 安装nodejs
RUN apt-get update -y

RUN apt-get install nodejs npm -y
RUN npm cache clean --force
RUN npm config set registry https://mirrors.xxx.com/repository/NPM/
RUN npm install -g gitbook-cli@2.1.2

RUN mkdir -p /usr/local/gitbook/data
RUN mkdir -p /usr/local/gitbook/tmp/docs
RUN chown -R gitlab-runner:gitlab-runner /usr/local/gitbook/tmp
COPY book.js /usr/local/gitbook/tmp/book.js
COPY custom.css /usr/local/gitbook/tmp/custom.css
COPY favicon.ico /usr/local/gitbook/tmp/favicon.ico
COPY auto_generate_summary.sh /usr/local/gitbook/auto_generate_summary.sh
```

### docker启停脚本

`docker-compose.yml`用于启动`GitLab Runner`与`Nginx`容器

```yaml
version: "3"
services:
 gitbook:
   privileged: true
   image: gitbook_custom:v1.0
   volumes:
     - $PWD/data:/usr/local/gitbook/data:rw
   restart: always
   container_name: gitbook-custom
 nginx:
   privileged: true
   image: nginx:1.22
   ports:
     - "3000:80"
   volumes:
     - $PWD/data:/usr/share/nginx/html
   restart: always
   container_name: gitbook-nginx
```

### 自动构建脚本

`.gitlab-ci.yml`是`GitLab Runner`构建时使用的文件，此文件规定了构建过程中要具体执行的步骤，只有此文件必须存在于目标文档对应的`GitLab`仓库，用于每次修改文档时触发自动构建

```yaml
variables:
  # 是否自动生成目录
  AUTO_SUMMARY: "true"

# 定义ci/cd 执行build流程
stages:
  - build

# build阶段执行的操作命令 
build:
  stage: build
  tags:
    - xxx-doc
  script:
    - echo "======== start build ========"
    - rm -rf /usr/local/gitbook/tmp/docs/*
    - cp -r * /usr/local/gitbook/tmp/docs/
    - mkdir -p /usr/local/gitbook/tmp/docs/styles
    - cd /usr/local/gitbook/tmp
    - cp custom.css  /usr/local/gitbook/tmp/docs/styles/website.css
    - bash /usr/local/gitbook/auto_generate_summary.sh $PWD
    - gitbook build
    - rm -rf /usr/local/gitbook/data/*
    - cp -r /usr/local/gitbook/tmp/_book/*  /usr/local/gitbook/data/
    - cp -rf /usr/local/gitbook/tmp/favicon.ico  /usr/local/gitbook/data/gitbook/images/favicon.ico
```

### 自动注册脚本

`gitlab_runner_config_init.sh`用于注册`GitLab Runner`同时检测`GitBook`是否存在

```bash
#/bin/bash

#更新文件权限

FOLDER="/usr/local/gitbook/data"
docker exec -it gitbook-custom bash -c "chown -R gitlab-runner:gitlab-runner $FOLDER"
echo "===================完成$FOLDER数据目录权限修改====================="


# 删除gitlab runner
GITLAB_URL='http://aeectss.xxx.local:1800/'
echo '===================开始删除旧的gitlab runner====================='
PRIVATE_TOKEN='5JZxxYzapBB_zfze5c2_'
PROJECT_ID=57
ids=$(curl --header "PRIVATE-TOKEN:${PRIVATE_TOKEN}" "${GITLAB_URL}/api/v4/runners"  |jq -r '.[]|"\(.id)"')
for id in $ids; do
  eval "curl --request DELETE --header 'PRIVATE-TOKEN:${PRIVATE_TOKEN}' '${GITLAB_URL}/api/v4/runners/${id}'"
  printf "\033[32m=====================删除id为${id}的runner======================\033[0m\n"
done


# 注册gitlab runner
CUR_DATE=$(date "+%Y-%m-%d %H:%M:%S")
TOKEN='M9VwuFfu9E42mb5QRx7M'
TAG_LIST='xxxxx-doc'
DESC='xxxxx文档撰写'
echo '===================开始注册gitlab runner====================='
sudo docker exec -it gitbook-custom \
gitlab-runner register --non-interactive \
--url "${GITLAB_URL}"  \
--registration-token "${TOKEN}" \
       --executor "shell" \
       --tag-list "${TAG_LIST}" \
--description "${DESC} -- 添加时间:${CUR_DATE}"


# 安装gitbook
echo '===================开始检查gitbook====================='
check=$(docker exec -it --user gitlab-runner gitbook-custom gitbook ls)

if [[ $check == *"no versions installed"* ]]; then
  echo "gitbook在gitlab-runner用户下没有安装"
  command="npm config set registry https://mirrors.xxx.com/repository/NPM/"
  command="${command};gitbook fetch"
  command="${command};cd /usr/local/gitbook/tmp"
  command="${command};npm install gitbook-plugin-mermaid-fox"
  command="${command};gitbook install"
  #command="${command};cp /usr/local/gitbook/tmp/node_modules/prismjs/components/prism-docker.js /usr/local/gitbook/tmp/node_modules/prismjs/components/prism-dockerfile.js"
  command="docker exec -it --user gitlab-runner gitbook-custom bash -c '$command'"
  echo "=========================要执行的命令======================="
  printf "\033[32m${command}\033[0m\n"
  echo "============================================================"
  eval $command
else
  printf "\033[32mGitbook已经安装，相关检查结果如下:\033[0m\n"
  echo "$check"
fi

printf "\033[32m===================gitlab runner初始化完毕!=====================\033[0m\n"
```

### 自动生成目录

`auto_generate_summary.sh`用于自动生成目录，`GitBook`左侧的目录树依赖于此脚本的生成结果，此文件会在`GitLab Runner`构建阶段执行

```bash
#!/bin/bash
# 先清空文件
echo "" >  docs/SUMMARY.md
# 子文件夹读取的递归函数
dirReader(){
    # $1 文件绝对路径
    # $2 章节名称
    # $3 缩进
    # 循环处理该文件夹下的文件
    for item in "${1}"/*
    do
        # 判断是否是文件夹且不是图片文件夹
        if [ -d "$item" ]  &&  ! [[ "${item}" =~ assets$ ]]
        then
            # 判断该文件夹是否存在README.md文件
            if [ ! -f "$item/README.md" ]
            then 
                # 如果没有就创建README.md
                echo "" > "$item/README.md"
            fi
            # 获取文件名称
            beginPos=`expr ${#1} + 1`
            endPos=`expr ${#item} - ${#1}`
            dirName=${item:$beginPos:$endPos}
            echo ${item}
            # 写入目录文件
            echo "$3+ [$dirName]($2/$dirName/README.md)" >> docs/SUMMARY.md
            # 递归处理该文件夹
            dirReader "$item" "$2/$dirName" "$3  "
        # 判断该文件是否是.md结尾 并且不是README.md
        elif [ -f "$item" ] && [ "${item##*.}" == 'md' ] 
        then
            beginPos=`expr ${#1} + 1`
            endPos=`expr ${#item} - ${#1} - 4`
            dirName=${item:$beginPos:$endPos}
            # 写入目录文件
            if[ "$dirName" != 'README' ]
            then
                echo ${item}
                echo "$3+ [$dirName]($2/$dirName.md)" >> docs/SUMMARY.md
            fi
        fi
    done
}
# 读取当前路径
dirs="${PWD}/docs/"
echo "- [主页](README.md)" >  docs/SUMMARY.md
# 循环处理该文件夹下的文件dir
for dir in docs/*
do
    # 截取掉dir路径名称里的 docs/
    startPos='5'
    length=`expr ${#dir} - 5`
    dir=${dir:$startPos:$length}
    # 判断是否是文件夹且不是图片文件夹
    if [ -d "${dirs}/${dir}" ] && ! [[ "${dir}" =~ assets$ ]] && ! [[ "${dir}" =~ styles$ ]] 
    then 
        # 判断该文件夹是否存在README.md文件
        if [ ! -f "$dirs/$dir/README.md" ]
        then 
        	echo "" > "$dirs/$dir/README.md"
        fi
        # 写入目录文件
        echo "+ [$dir]($dir/README.md)" >> docs/SUMMARY.md
        # 处理文件夹内部的文件
        dirReader "$dirs$dir" "$dir" "  "
    # 判断该文件是否是.md结尾 并且不是SUMMARY.md 和 README.md
    elif  [ "${dir##*.}" == 'md' ] && [ "${dir}" != "SUMMARY.md" ] && [ "${dir}" != 'README.md' ]
    then
    # 写入目录文件
    	echo "+ [$dir]($dir)" >> docs/SUMMARY.md
    fi
done
```

### 目录分类显示

![GitBook目录分类显示](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-menu-category.png "GitBook目录分类显示")

若需要在生成目录时能自动进行如上图所示的分类展示，则可将上述脚本修改为类似如下

```bash
#!/bin/bash
: '
此脚本用于生成目录时可根据目录类别分成多个子类，要分类的顶层文件夹记录在special_tags数组变量中
'
# 先清空文件
echo "" >  docs/SUMMARY.md
# 子文件夹读取的递归函数
dirReader(){
	# $1 文件绝对路径
	# $2 章节名称
	# $3 缩进
	# 循环处理该文件夹下的文件
	for item in "${1}"/*
	do	
		# 判断是否是文件夹且不是图片文件夹
		if [ -d "$item" ]  &&  ! [[ "${item}" =~ assets$ ]]
		then
			# 判断该文件夹是否存在README.md文件
			if [ ! -f "$item/README.md" ]
			then 
				# 如果没有就创建README.md
				echo "" > "$item/README.md"
			fi
			# 获取文件名称
			beginPos=`expr ${#1} + 1`
			endPos=`expr ${#item} - ${#1}`
			dirName=${item:$beginPos:$endPos}
			echo ${item}
			# 写入目录文件
			echo "$3+ [$dirName]($2/$dirName/README.md)" >> docs/SUMMARY.md
			# 递归处理该文件夹
			dirReader "$item" "$2/$dirName" "$3  "
		# 判断该文件是否是.md结尾 并且不是README.md
		elif [ -f "$item" ] && [ "${item##*.}" == 'md' ] 
		then
			beginPos=`expr ${#1} + 1`
			endPos=`expr ${#item} - ${#1} - 4`
			dirName=${item:$beginPos:$endPos}
			# 写入目录文件
			if	[ "$dirName" != 'README' ]
			then
				echo ${item}
				echo "$3+ [$dirName]($2/$dirName.md)" >> docs/SUMMARY.md
			fi
		fi
	done
}

baseReader(){
    # 需要另外新开一个分类
	special_tags=("VDE相关" "云授权平台")
    regex=$(IFS='|'; echo "${special_tags[*]}")
	special_dirs=()

	# 读取当前路径
	dirs="${PWD}/docs/"
	echo "- [主页](README.md)" >  docs/SUMMARY.md
	# 循环处理该文件夹下的文件dir
	for dir in docs/*
	do
		# 截取掉dir路径名称里的 docs/
		startPos='5'
		length=`expr ${#dir} - 5`
		dir=${dir:$startPos:$length}
		if echo "$dir" | egrep -iq "$regex"  ; then
		   special_dirs+=("$dir")
		else
		   baseWrite "$dirs" "$dir"
		fi
	done
	
	# 处理特殊类别的文件
	if [[ -n "$special_dirs" ]]; then
	    echo $'\n#\n' >> docs/SUMMARY.md
	fi
	for dir in "${special_dirs[@]}"; do
	    baseWrite "$dirs" "$dir"
	done 
	
}

baseWrite(){
    dirs=$1
	dir=$2
	# 判断是否是文件夹且不是图片文件夹
	if [ -d "${dirs}/${dir}" ] && ! [[ "${dir}" =~ assets$ ]] && ! [[ "${dir}" =~ styles$ ]] 
	then 	
		# 判断该文件夹是否存在README.md文件
		if [ ! -f "$dirs/$dir/README.md" ]
		then 
			echo "" > "$dirs/$dir/README.md"
		fi
		
		# 写入目录文件
		echo "+ [$dir]($dir/README.md)" >> docs/SUMMARY.md
		# 处理文件夹内部的文件
		dirReader "$dirs$dir" "$dir" "  "
	# 判断该文件是否是.md结尾 并且不是SUMMARY.md 和 README.md
	elif  [ "${dir##*.}" == 'md' ] && [ "${dir}" != "SUMMARY.md" ] && [ "${dir}" != 'README.md' ]
	then
			# 写入目录文件
			echo "+ [$dir]($dir)" >> docs/SUMMARY.md
	fi
}

baseReader
```

### 自定义样式

`custom.css`是自定义样式文件，当对`GitBook`的某些样式不满意时，可用自定义`CSS`文件来覆盖

```css
.book .book-body .page-wrapper .page-inner {
  max-width: 1200px !important;
}

.markdown-section img:not(.emoji) {
    max-width: 100%;
    border: 1px dashed #a9a4a4;
}

table > thead > tr > th {
   text-align:center !important;
}


.markdown-section code:not([class^="lang-"]) {
    padding: 3px 5px;
    margin: 0;
    font-size: .85em;
    color: #c7254e;
    border-radius: 4px;
}

small {
    font-size: 80% !important;
}

section  {
    width:100%;
}
h1 {
  color: #2674BA;
}
h2 {
  color: #0099CC;
}
h3 {
  color: #F77A0B;
}
h4 {
  color: #662D91;
}
h5 {
  color: #444444;
}
th {
  background-color: #2674BA;
  color: white;
}
```

### GitBook配置文件

`book.js`是`GitBook`的配置文件，包含全局配置与各种插件

```javascript
module.exports = {
    // markdown文档所在路径
    root: './docs',
    // 项目标题
    title: 'xxxxx平台文档',
    // 版本
    gitbook: '3.2.3',
    language: 'zh-hans',
    // 插件
    plugins: [
        '-highlight',
        'prism-codetab-fox',
        'advanced-emoji',
        'mermaid-fox', // 图表
        'graph',
        'flowchart-fox',
        'chart',
        'popup',
        'accordion',
        'katex',
        '-search', //搜索
        'search-pro', //中文搜索
        'hide-element', //元素隐藏
        '-lunr', //索引
        'chapter-fold', //导航目录折叠
        'code', //代码复制按钮
        'splitter', //侧边栏宽度可调节
        'expandable-chapters-small', //目录收起
        'tbfed-pagefooter', //页面添加页脚
        'ancre-navigation', //页内导航回到顶部
        '-sharing', 'sharing-plus', //分享链接
        'theme-comscore', //主题
        'favicon', //图标
        'flexible-alerts' // 警告
    ],
    // 插件配置
    pluginsConfig: {
        'prism': {
            'css': [
                'prismjs/themes/prism-solarizedlight.css'
            ],
            "lang": {
                "dockerfile": "docker"
            },
            "ignore": [
                "mermaid",
                "eval-js"
            ]
        },
        'hide-element': {
            'elements': ['.gitbook-link'] //需要隐藏的元素，可以通过浏览网页找到该class
        },
        'tbfed-pagefooter': {
            'copyright': 'Copyright &copy xxx.com 2023-2033',
            'modify_label': '修订时间：',
            'modify_format': 'YYYY-MM-DD HH:mm:ss'
        },
        'sharing': {
            'all': []
        },
        "chart": {
            "type": "c3"
        },
        "chapter-fold": {},
        "favicon": {
            "shortcut": "/favicon.ico",
            "bookmark": "/favicon.ico"
        },
        'flowchart': {
            "arrow-end": "block",
            "element-color": "black",
            "fill": "white",
            "flowstate": {
                "approved": {
                    "fill": "#58C4A3",
                    "font-size": 12
                },
                "current": {
                    "fill": "#008080",
                    "font-color": "white",
                    "font-weight": "bold"
                },
                "future": {
                    "fill": "#8A624A"
                },
                "success": {
                    "fill": "#0B6623"
                },
                "invalid": {
                    "fill": "#6c5696"
                },
                "past": {
                    "fill": "#CCCCCC",
                    "font-size": 12
                },
                "select": {
                    "fill": "#E1AD01",
                    "font-size": 12
                },
                "rejected": {
                    "fill": "#C45879",
                    "font-size": 12
                },
                "request": {
                    "fill": "#569656"
                },
                'yellowgreen': {
                    "fill": "yellowgreen"
                },
                'failed': {
                    "fill": "#800020"
                }
            },
            "font-color": "black",
            "font-size": 14,
            "line-color": "black",
            "line-length": 50,
            "line-width": 1.3,
            "scale": 1,
            "symbols": {
                "end": {
                    "class": "end-element",
                    "font-color": "white",
                    "font-weight": "bold"
                },
                "start": {
                    "fill": "#8c2106",
                    "font-color": "white",
                    "font-weight": "bold"
                }
            },
            "text-margin": 10,
            "width": 1,
            "x": 0,
            "y": 0,
            "yes-text": "是",
            "no-text": "否"
        }
    },
    variables: {
        "styles": {
            "website": "./styles/website.css",
        }
    }
};
```

[^1]: 可根据实际要求通过`-lunr`来屏蔽对应的索引插件