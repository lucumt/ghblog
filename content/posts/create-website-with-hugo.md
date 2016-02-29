+++

author = "卢运强"
categories = ["Go","Hugo","Github Pages"]
date = "2016-02-27T22:23:37+08:00"
description = "Blog of Rosen Lu"
keywords = ["Github Pages","Go","Hugo"]
title = "利用Github Pages和基于Go的Hugo搭建个人博客"
nocomment = true

+++

一直以来都想拥有一个属于自己的博客，前段时间在学习`Go` ，于是利用`Hugo` 和`Github Pages` 搭建了一个简易的个人博客，先简单记录下。 

[//]:(设置前面的内容为summary)
<!--more-->
 
# 环境准备
* [Go](https://golang.org/)1.4+
* [Hugo](https://gohugo.io)v0.14+
* [Github Pages](https://pages.github.com/)账号
* [GoDaddy](https://www.godaddy.com)域名 

# 过程概要
## 在Github上创建一个自己的项目
参考`[Github Pages]` 官网首页的说明，按照提示步骤依次操作即可。在此过程中，有两个步骤需要我们进行选择：  
1. 创建**User or origanization site**或**Project site**,我个人选择的是**Project site**。
![png_1](https://ooo.0o0.ooo/2016/02/29/56d46b8727135.png)
2. 若选择的是**Project site**,则会让选择**Generate a site**或**Start from scratch**，个人选择**Start from scratch**从头开始搭建。
![png_2.png](https://ooo.0o0.ooo/2016/02/27/56d1c72e8b3ba.png)
当然个人的爱好不同，不一定非得选择**Project site**和**Start from scratch**，这不是本文的重点，只要能按照`Github Pages`的提示成功搭建自己的静态博客即可。

## 利用Hugo作为博客生成器
由于`Github Pages` 只支持静态的html页面托管，所以需要采用`Jekyll` 、`Logdown` 等静态博客生成器来快速生成HTML页面，避免纯手动编写时的费时费力。

由于自己近期一直在学习`Go`，为了加深自己对于`Go`的运用，于是便选择`Hugo` 作为自己的博客生成器。`Hugo` 是一个基于`Go]`开发的静态生成器，它采用*[Markdown](https://zh.wikipedia.org/zh-cn/Markdown)* 语法来编写博客生成，然后生成相应的HTML页面。

### 安装Go
访问[Golang下载页](https://golang.org/dl/)根据自己电脑的操作系统选择是Linux版本或Windows版本，同时注意是选择32位还是64位，一定要与自己的操作系统相匹配。
以我自己的64位win7系统为例，安装过程如下：

* 下载**go1.4.2.windows-amd64.msi**
* 双击安装，默认是安装在C盘下，由于windows操作系统的特性，我通常不倾向于安装在C盘，故需要设置`PATH`、`GOPATH`和`GOROOT`这三个环境变量，我自己把`Go`安装在*D:\code\go*下，这三个变量相应的设置为:

>PATH='D:\code\go\bin';%PATH%  
>GOPATH='D:\code\gopath'  
>GOROOT='D:\code\go'

其中**PATH**变量的作用主要是为了将`Go`的各种命令加到cmd中。**GOROOT**则是当我们将`Go`安装在我们的自定义目录时，需要制定该安装目录，若安装在默认目录**C:\go**下则不用设置**GOROOT**目录，官方文档原文为*GOROOT must be set only when installing to a custom location.* 比较难懂的是**GOPATH**,官方文档上的说明为 *Create a directory to contain your workspace, $HOME/work for example, and set the GOPATH environment variable to point to that location.* 据此我们可以将**GOPATH**理解为工作目录，它主要用来存放`Go`代码编译之后的可执行文件，其位置可以随意设置，只要不与**GOROOT**相同即可。

* 安装完成之后，重新打开cmd窗口，输入**go version**之后按Enter键，若出现如下信息，则表示`Go`安装成功:

>C:\Users\Administrator>go version  
>go version go1.4.2 windows/amd64

### 安装Hugo
`Hugo`的安装过程与`Go`的类似.

* 首先在[Hugo下载页](https://github.com/spf13/hugo/releases)根据自己操作系统的类型和位数下载相应的安装包，然后设置对应的**PATH**环境变量即可。我的安装在 *D:\program\hugo* 所以相应的环境变量设置为

> PATH='D:\program\hugo';%PATH%

* 重新打开cmd窗口，输入**hugo version**，若出现如下信息，则表示`Hugo`安装成功：

>C:\Users\Administrator>hugo version
>Hugo Static Site Generator v0.14 BuildDate: 2015-05-26T01:29:16+08:00

关于`Hugo`的基本操作命令，可以参见[Hugo快速入门](https://gohugo.io/overview/quickstart/)，此处不再详述。

## 在Github Pages上托管Hugo

### 安装命令
最权威的信息还是来源于官方文档，可以参见[在Github Pages上托管Hugo](https://gohugo.io/tutorials/github-pages-blog/),主要的`Git`操作命令如下：


{{< highlight ruby >}}# Create a new orphand branch (no commit history) named gh-pages
git checkout --orphan gh-pages

# Unstage all files
git rm --cached $(git ls-files)

# Grab one file from the master branch so we can make a commit
git checkout master README.md

# Add and commit that file
git add .
git commit -m "INIT: initial commit on gh-pages branch"

# Push to remote gh-pages branch
git push origin gh-pages

# Return to master branch
git checkout master

# Remove the public folder to make room for the gh-pages subtree
rm -rf public

# Add the gh-pages branch of the repository. It will look like a folder named public
git subtree add --prefix=public git@github.com:spencerlyon2/hugo_gh_blog.git gh-pages --squash

# Pull down the file we just committed. This helps avoid merge conflicts
git subtree pull --prefix=public git@github.com:spencerlyon2/hugo_gh_blog.git gh-pages

# Run hugo. Generated site will be placed in public directory (or omit -t ThemeName if you're not using a theme)
hugo -t ThemeName


# Add everything
git add -A

# Commit and push to master
git commit -m "Updating site" && git push origin master

# Push the public subtree to the gh-pages branch
git subtree push --prefix=public git@github.com:spencerlyon2/hugo_gh_blog.git gh-pages{{< /highlight >}}

### 说明信息

* 上述命令的核心思想把源代码放在**master**上，把生成的静态html代码通过**substree**命令放到**branch**上；
* `Git`主干的名字可以随便取，但是分支的名字必须为`gh-pages`；
* 上述命令针对的是从头到尾开始创建一个`Git`项目然后提交，如果之前已经有`gh-pages`项目了，则可以从 *git subtree add --prefix=public git@github.com:spencerlyon2/hugo_gh_blog.git gh-pages --squash* 这句代码开始执行；

<--待续-->