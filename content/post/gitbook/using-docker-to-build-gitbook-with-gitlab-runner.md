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

由于`Docsify`采用单页渲染的方式，在初次打开是特别慢，近期将团队内部的知识库工具从`Docsify`迁移到基于`Docker`搭建的`GitBook`，在这其中踩了一些坑，简单记录下。

<!--more-->

# 背景

## 技术选型

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

## 使用流程

由于文档同代码一样都是采用`GitLab`管理的，为了实现在文档更新时能够自动化的部署，准备采用`GitLab Runner`+`GitBook`+`Nginx`的方案，整体流程图如下：

![GitBook结合GitLab Runner使用流程](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-gitlab-usage-flow.png "GitBook结合GitLab Runner使用流程")

在实际使用时，相关用户只需要在按照常规的流程在本地基于`Markdown`格式编写文档并提交到`GitLab`中，之后`GitLab Runner`会自动化的进行文档构建与文档生成，依赖于宿主物理机的配置和文档数量，整个过程通常不超过5秒钟，之后在浏览器中刷新即可查看生成的`HTML`静态文件。

# 环境搭建

出于简化使用流程与加快构建速度的考虑，将`GitLab Runner`与`GitBook`都放置到同一个容器中。在`GitLab Runner`的`Docker`容器中安装完毕`GitBook`环境后，还需将`GitLab Runner`注册到对应的`GitLab`仓库中，可通过`Shell`脚本将此过程固化下来，整个环境搭建流程如下：

![GitBook安装流程](/blog_img/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/gitbook-install-flow.png "GitBook安装")

## 安装过程

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

# 使用展示

# 文件&脚本

* `Dockerfile`文件，用于构建同时包含`GitLab Runner`与`GitBook`的容器

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

* docker-compose.yml文件，用于启动`GitLab Runner`与`Nginx`容器

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

* `.gitlab-ci.yml`文件，只有此文件必须存在于目标文档对应的`GitLab`仓库，用于每次修改文档时触发自动构建

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

* `gitlab_runner_config_init.sh`，用于注册`GitLab Runner`同时检测`GitBook`是否存在

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

* `auto_generate_summary.sh`，自动生成目录

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

* `custom.css`，自定义样式文件

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

* `book.js`，`GitBook`的配置文件，包含全局配置与各种插件

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

  