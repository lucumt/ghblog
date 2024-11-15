---
title: "利用Hugo来替代GitBook实现文档展示"
date: 2024-09-06T10:26:52+08:00
lastmod: 2024-09-06T10:26:52+08:00
draft: false
keywords: ["hugo","gitbook","文档展示"]
description: "简要记录如何利用Hugo来搭建并生成类似GitBook的展示效果，给文档管理提供另外一种选择"
tags: ["hugo","gitbook"]
categories: ["工具使用","个人博客"]
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
codeTabSeperator: "::"

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

基于[Hugo](https://gohugo.io)中的[hugo-book](https://github.com/alex-shpak/hugo-book)主题简要记录搭建并生成类似`GitBook`的展示效果，以便在需要进行文档管理与展示时，可以多一种选择。

<!--more-->

## 背景

部门内部使用[GitBook](https://www.gitbook.com)进行知识库的管理已经有一段时间[^1]，并且在此期间个人也积累了[一些经验](/tags/gitbook/)，但由于其实基于`Nodejs`开发以及开源的`v2`版本不再维护的原因[^2]，导致其构建速度、环境搭建、插件开发等方面都存在一些不便之处。



由于自己一直使用`Hugo`进行博客编写，而`Hugo`是一款知名的博客编写工具，其生态圈十分繁荣，为了给部门和自己提供更多的选择，个人计划采用`Hugo`实现类似的功能作为可选的替代品。在其官方网站的[主题列表](https://themes.gohugo.io)中发现`hugo-book`的显示效果和`GitBook`类似，简单测试后基本满足要求。

## 搭建与配置

### 环境搭建

依次执行下述命令

```bash
git clone https://github.com/alex-shpak/hugo-book.git
cd hugo-book

# 删除demo页
rm -rf exampleSite

# 暂定不支持多语言,只创建content目录即可
mkdir -p content/docs && cd content

# 创建简单的测试页面
echo "I am defult page" >> _index.md
echo "Hello world A" >> docs/index1.md
echo "Hello world B" >> docs/index2.md

# 创建配置文件
echo "title = '个人笔记'" >> config.toml

# 启动测试
hugo server
```

若一切执行顺利，终端会出现类似如下输出

 ![hugo网站启动正常](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo_server_start_success.png "hugo网站启动正常")

此时在浏览器中输入`http://127.0.0.1:1313`会有如下输出，默认主页展示的内容为`content/_index.md`中的内容，同时在页面左侧有名为`index1`和`index2`的菜单链接，其对应于我们在`content/docs`目录下创建的文件名

 ![系统主页展示](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo-book-home-page.png "系统主页展示")

点击链接后会显示对应文件的内容，支持`hugo-book`的环境基本搭建完毕。

 ![系统页面切换](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo-page-menu-switch.png "系统页面切换")

{{% admonition success "总结" false %}}

* 默认的页面内容位于`content/_index.md`文件中
* 除了默认页面之外，其余的内容都需位于`content/docs`目录下

{{% /admonition %}}

### 页面别名

虽然按照前述步骤已经能正常工作，但左侧的菜单名称默认是文件名称，使用起来不直观，需要将其设置为具有含义的名称，虽然可通过修改文件名称的方式来实现，但当标题包含中文或特殊字符时，可能会有显示问题。

`hugo-book`提供了名为`title`的属性来自定义页面显示内容，可将对应页面修改如下

{{% codetabs %}}  

  ```bash::index1.md
---
title: "第一个页面"
---
  ```

```bash::index2.md
---
title: "第二个页面"
---
```

{{% /codetabs %}}  

之后浏览器中的显示效果如下

 ![自定义菜单名称](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo-book-with-custom-menu-title.png "自定义菜单名称")

### 目录层级

在实际的知识库文档管理中通常，页面数通常很多，需要对齐进行归类管理，`hugo-book`默认也提供了这方面的支持。

创建类似如下的目录，并添加对应的文件，在文件中修改`title`属性，在文件夹下创建`_index.md`文件并修改其`title`属性为期望的名称，根据实际情况添加`bookCollapseSection: true`属性。

```bash
D:\code\githubCode\hugo-book\content>tree /F
文件夹 PATH 列表
卷序列号为 33B9-4C35
D:.
│  _index.md
│
└─docs
    │  index1.md
    │  index2.md
    │
    ├─algorithm
    │      dynamic-program.md
    │      sorted-stack.md
    │      _index.md
    │
    ├─code
    │  │  _index.md
    │  │
    │  ├─golang
    │  │      _index.md
    │  │
    │  ├─java
    │  │      print-a-b-c.md
    │  │      _index.md
    │  │
    │  └─python
    │          pytorch-tutorial.md
    │          _index.md
    │
    └─db
            _index.md
```

操作完毕后，显示效果如下

 ![具有多层级结构的菜单](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo-book-with-multiple-menu-level.png "具有多层级结构的菜单")

{{% admonition success "总结" false %}}

* 目录名称默认为文件夹名称，但可通过在对应目下的`_index.md`文件设置`title`属性来实现
* 可通过在文件夹下对应目下的`_index.md`文件中添加`bookCollapseSection: true`来控制目录是否支持展开/折叠

{{% /admonition %}}

### 页面顺序

可通过修改页面或`_index.md`中的`weight`属性来调整统一层级下的页面和目录结构的显示顺序，实现个性化的顺序显示。

将签署代码中的相关文件头部修改如下

{{% codetabs %}}  

  ```bash::index1.md
---
title: "第一个页面"
weight: 2
---
  ```

```bash::index2.md
---
title: "第二个页面"
weight: 1
---
```

```bash::code/_index.md
---
bookCollapseSection: true
weight: 3
title: 编程
---
```

```bash::algorithm/_index.md
weight: 4
title: 算法
```

```bash::db/_index.md
bookCollapseSection: true
weight: 5
title: 数据库
```

{{% /codetabs %}}  

修改后的展示效果类似如下

 ![自定义页面和目录的展示顺序](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo-page-with-custom-order.png "自定义页面和目录的展示顺序")

实际上在`archetypes`下的`posts.md`文件中已经提供默认的配置模板，当我们采用`hugo new`命令创建页面时，`hugo`会将Z和谐默认参数带入，只需根据实际情况修改即可。

```bash
---
title: "{{ .Name | humanize | title }}"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---
```

### 代码高亮

默认的代码高亮显示类似如下

 ![默认的代码高亮样式](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo-book-default-code-highlight.png "默认的代码高亮样式")

个人不太喜欢其风格，可在`config.toml`中添加类似如下配置修改为自己喜欢的样式

```toml
[markup]
  [markup.goldmark.renderer]
    unsafe = true
  [markup.tableOfContents]
	startLevel = 1
  [markup.highlight]
    codeFences = true
    guessSyntax = true
    hl_Lines = ""
    lineNoStart = 1
    lineNos = true
    lineNumbersInTable = false
    noClasses = true
    style = "perldoc"
    tabWidth = 4
```

修改后的显示效果如下

 ![修改后的代码高亮样式](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo-book-updated-code-highlight.png "修改后的代码高亮样式")

## 样例展示

基于前述操作步骤以及[创建多个GitHub Pages并且利用GoDaddy分别配置子域名](/post/github/create-multiple-github-pages-and-config-godaddy-subdomain/)中的说明，自己搭建了一个展示网站[**https://tech.lucumt.info**](https://tech.lucumt.info/),展示效果如下，源码位于[https://github.com/lucumt/tech](https://github.com/lucumt/tech)

 ![个人网站展示效果-1](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo-book-for-lucumt-1.png "个人网站展示效果-1")

 ![个人网站展示效果-2](/blog_img/hugo/using-hugo-to-show-document-like-gitbook/hugo-book-for-lucumt-2.png "个人网站展示效果-2")

[^1]: 主要是基于开源版本的`v2`版本
[^2]: 参见[Deprecation warning](https://github.com/GitbookIO/gitbook-cli?tab=readme-ov-file#%EF%B8%8F-deprecation-warning)