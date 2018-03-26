+++

author = "飞狐"
categories = ["Go编程","个人博客"]
tags=["Go","Hugo","Github Pages"]
date = "2016-02-27T22:23:37+08:00"
description = "Blog of Rosen Lu"
keywords = ["Github Pages","Go","Hugo"]
title = "利用Github Pages和基于Go的Hugo搭建个人博客"

+++

一直以来都想拥有一个属于自己的博客，前段时间在学习`Go` ，于是利用`Hugo` 和`Github Pages` 搭建了一个简易的个人博客，先简单记录下。 

[//]:(设置前面的内容为summary)
<!--more-->

## 环境准备
* [Go](https://golang.org/)1.4+
* [Hugo](https://gohugo.io)v0.14+
* [Github](https://github.com/)账号
* [GoDaddy](https://www.godaddy.com)域名 

## 过程概要
### 在Github上创建一个自己的项目
1. 在 *Github* 上创建一个项目，本文中该项目名为 *blog*  
	![创建Github项目](/blog_img/create-website-with-hugo/create-github-repository.png "创建Github项目")
2. 由于`Github Pages`强制要求在托管博客时该项目必须有一个名为 *gh-pages* 的分支，所以要预先给该项目创建一个名为 *gh-pages* 的分支
3. 在Github中打开 *blog* 项目主页面，点击 *Settings* 按钮  
	![打开项目配置界面](/blog_img/create-website-with-hugo/open-github-project-settings.png "打开Github项目设置界面")  
	在 *Github Pages* 这个区域可以看见本项目的发布链接为 *https\://fox321.github.io/blog/* ，点击该链接可以访问该项目对应的静态页面   
	![查看项目发布链接](/blog_img/create-website-with-hugo/check-github-project-address.png "查看项目发布链接")

### 利用Hugo作为博客生成器
由于 *Github Pages* 只支持静态的html页面托管，所以需要采用`Jekyll` 、`Logdown` 等静态博客生成器来快速生成HTML页面，避免纯手动编写时的费时费力。由于自己近期一直在学习 *Go*，为了加深自己对于 *Go* 的运用，于是便选择 *Hugo*  作为自己的博客生成器。 *Hugo* 是一个基于 *Go* 开发的静态生成器，它采用*[Markdown](https://zh.wikipedia.org/zh-cn/Markdown)* 语法来编写博客生成，然后生成相应的HTML页面。

#### 安装Go
访问[Golang下载页](https://golang.org/dl/)根据自己电脑的操作系统选择是Linux版本或Windows版本，同时注意是选择32位还是64位，一定要与自己的操作系统相匹配。以我自己的64位win7系统为例，安装过程如下：

* 下载**go1.4.2.windows-amd64.msi**
* 双击安装，默认是安装在C盘下，由于windows操作系统的特性，我通常不倾向于安装在C盘，故需要设置`PATH`、`GOPATH`和`GOROOT`这三个环境变量，我自己把 *Go* 安装在 *D:\code\go* 下，这三个变量相应的设置为:

	```
	PATH='D:\code\go\bin';%PATH%  
	GOPATH='D:\code\gopath'  
	GOROOT='D:\code\go'
	```

其中**PATH**变量的作用主要是为了将 *Go* 的各种命令加到cmd中。**GOROOT**则是当我们将 *Go* 安装在我们的自定义目录时，需要制定该安装目录，若安装在默认目录**C:\go**下则不用设置**GOROOT**目录，官方文档原文为*GOROOT must be set only when installing to a custom location.* 比较难懂的是**GOPATH**,官方文档上的说明为 *Create a directory to contain your workspace, $HOME/work for example, and set the GOPATH environment variable to point to that location.* 据此我们可以将**GOPATH**理解为工作目录，它主要用来存放 *Go* 代码编译之后的可执行文件，其位置可以随意设置，只要不与**GOROOT**相同即可。

* 安装完成之后，重新打开cmd窗口，输入**go version**之后按Enter键，若出现如下信息，则表示 *Go* 安装成功:

	```
	C:\Users\Administrator>go version  
	go version go1.4.2 windows/amd64
	```

#### 安装Hugo
*Hugo* 的安装过程与 *Go* 的类似.

* 首先在[Hugo下载页](https://github.com/spf13/hugo/releases)根据自己操作系统的类型和位数下载相应的安装包，然后设置对应的**PATH**环境变量即可。我的安装在 *D:\program\hugo* 所以相应的环境变量设置为

	```
	PATH='D:\program\hugo';%PATH%
	```

* 重新打开cmd窗口，输入**hugo version**，若出现如下信息，则表示 *Hugo* 安装成功：

	```
	C:\Users\Administrator>hugo version
	Hugo Static Site Generator v0.14 BuildDate: 2015-05-26T01:29:16+08:00
	```

关于 *Hugo* 的基本操作命令，可以参见[Hugo快速入门](https://gohugo.io/overview/quickstart/)，此处不再详述。


### 在Github Pages上托管Hugo

#### 安装命令
虽然官方文档[在Github Pages上托管Hugo](https://gohugo.io/tutorials/github-pages-blog/)上有相应的说明,个人总感觉其说明信息不够详细，故将自己的实现过程记录如下：

1. 将之前在Github上创建的 *blog* 项目clone到本地目录   
    ![将项目下载到本地](/blog_img/create-website-with-hugo/clone-github-repository.png "将项目下载到本地")  
2. 切换到 *blog* 目录并利用`hugo new site`命令创建一个名为 *person_blog* 的 *Hugo* 站点，然后将其内容移入到 *blog* 目录下并删除 *person_blog* 目录   
	![创建一个Hugo站点](/blog_img/create-website-with-hugo/create-hugo-site-in-repository.png "创建一个Hugo站点")
3. 利用`hugo new`命令创建一个md文件用于存储我们的第一篇博客  
	![创建一个Hugo页面](/blog_img/create-website-with-hugo/create-hugo-page.png "创建一个Hugo页面")
4. 在 *blog* 目录下创建一个名为 *themes* 的文件夹用于存储 *Hugo* 样式，并将自己选中的样式下载到本地  
	![下载Hugo样式](/blog_img/create-website-with-hugo/clone-hugo-theme.png "下载Hugo样式")
5. 输入`hugo server --theme=hugo-redlounge --buildDrafts`在本地启动Hugo，启动正常后命令行输出如下  
	![在本地启动Hugo](/blog_img/create-website-with-hugo/start-hugo-in-local.png "在本地启动Hugo")  
	此时在浏览器中输入`http://127.0.0.1:1313`会看到如下输出，该页面意味着本地博客创建成功，接下来要将其上传到`Github Pages`中托管    
	![在本地访问Hugo站点](/blog_img/create-website-with-hugo/visit-local-hugo-site.png "在本地访问Hugo站点")  
7. 我们需要将相关的链接地址修改为 *https\://fox321.github.io/blog* ，同时将端口号去掉，相关的命令为 `hugo server -D --theme=hugo-redlounge --baseUrl="https://fox321.github.io/blog" --appendPort=false`，运行截图如下
	![修改博客链接地址](/blog_img/create-website-with-hugo/update-hugo-site-url.png "修改博客链接地址")
8. 修改完链接地址之后，需要将生成的页面提交到 *Github* 中才能被访问，首先需要将页面提交到 *master* ，由于我是在Windows操作系统上进行的，而CMD对`Git`的支持不是很好，故从此步开始切换为在 *Git Bash* 进行相关操作    
	![提交到master](/blog_img/create-website-with-hugo/push-blog-to-github.png "提交到master")
9. 利用`subtree`命令将 *master* 中 *public* 目录下的内容同步到 *gh-pages* 目录下  
	![同步到gh-pages](/blog_img/create-website-with-hugo/push-blog-to-branch.png "同步到gh-pages")  
	此时访问该项目的设置页面，在 *Github Pages* 部分会看见如下信息  
    ![同步到gh-pages成功](/blog_img/create-website-with-hugo/push-blog-to-branch-success.png "同步到gh-pages成功")  
10. 访问 *https\://fox321.github.io/blog* ，出现如下页面，至此 *Hugo* 博客托管到 *Github Pages* 成功！  
	![访问Github Pages页面"](/blog_img/create-website-with-hugo/visit-github-pages-hugo-site.png "访问Github Pages页面")

#### 相关命令
1. 生成绑定到指定域名的页面 `hugo server -D --baseUrl="http://lucumt.info" --appendPort=false`
2. 将 *master* 的 *public* 目录同步到分支 `git subtree push --prefix=public git@github.com:fox321/blog.git gh-pages`

### 利用GoDaddy配置自定义域名
在[Using a custom domain with GitHub Pages](https://help.github.com/articles/using-a-custom-domain-with-github-pages/)中有详细的说明，我自己配置的时候主要是按照[Setting up an apex domain](https://help.github.com/articles/setting-up-an-apex-domain/)中的说明在[GoDaddy](https://www.godaddy.com/)上的说明来设置的。

1. 在public目录下创建一个名为 **CNAME** 的文件，并在该文件中写入我们要自定义的域名，我自己的域名为 *http://lucumt.info* ，故填入 *lucumt.info*  
!["创建CNAME文件"](/blog_img/create-website-with-hugo/create-cname-file.png "创建CNAME文件并添加域名")  
2. 登陆Godaddy，然后在页面右上角点击自己的用户名，出现如下图所示的页面，选择 *Manage My Domains*  
!["选择域名管理界面"](/blog_img/create-website-with-hugo/godaddy-choose-manage-page.png "选择域名管理界面")  
3. 选择完 *Manage My Domains* 之后会出现如下图所示的界面，选择 *Manage DNS*   
!["选择DNS管理界面"](/blog_img/create-website-with-hugo/godaddy-choose-manage-dns.png "选择DNS管理界面") 
4. 选择完 *Manage DNS* 之后会出现如下图所示的界面，点击 *Add* 按钮会出现下拉框让我们增加A记录  
!["DNS管理界面"](/blog_img/create-website-with-hugo/godaddy-dns-records-page.png "DNS管理界面")  
5. 在 *Type* 部分选择 **A** ，*Host* 部分选择 **@**， *Poinst to* 根据[Setting up an apex domain](https://help.github.com/articles/setting-up-an-apex-domain/)中的说明在 *GoDaddy* 中 *Configuring A records with your DNS provider* 部分的说明添加 **192.30.252.153** ,点击 *Save* 即完成一条A记录的添加  
!["增加A记录"](/blog_img/create-website-with-hugo/godaddy-dns-add-a-records.png "增加A记录")  
6. 再次点击 *Add* 添加，按照步骤5添加第二条A记录，除了 *Points to* 设置为 **192.30.252.154** 之外，其它的配置都相同
7. 在`Github`项目中点击 *Settings* 按钮查看 *Github Pages* 区域的设置信息，若出现类似如下图所示的设置信息，则表示我们自定义域名添加成功  
!["增加A记录"](/blog_img/create-website-with-hugo/github-pages-configuration-check.png "增加A记录")

至此利用 *GoDaddy* 来配置自自定义域名的过程完成，输入 *http://lucumt.info* 即可访问自己的博客！

PS:吐槽下让人觉得不爽的几个地方：

- 域名续费几乎没有折扣，所以建议第一次买的时间长一点。
- 取消订单功能消失了，现在只能打人工客服电话取消。

如果大家有 *GoDaddy* 之外更好的域名服务网站，欢迎给我留言，当然国内的域名服务商除外！

### 开启自定义域名的HTTPS访问

请参见本人写的另外一篇文章 **[将基于Github Pages的自定义域名博客迁移到Https](https://lucumt.info/posts/migrate-github-blog-from-http-to-https/)** ，此处不再详述。

<--待续-->