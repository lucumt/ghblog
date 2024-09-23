---
title: "创建多个GitHub Pages并且利用GoDaddy分别配置子域名"
date: 2024-05-20T09:44:44+08:00
lastmod: 2024-05-20T09:44:44+08:00
draft: false
keywords: ["GitHub","Github Pages","GoDaddy"]
description: "创建多个GitHub Pages并且利用GoDaddy分别配置子域名,实现域名使用价值的最大化同时节省开支"
tags: ["GitHub","Github Pages"]
categories: ["个人博客","其它"]
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

简要记录如何基于同一个`GitHub`账号创建多个[**GitHub Pages**](https://pages.github.com/)并基于[**GoDaddy**](https://www.godaddy.com/)配置多个子域名。

<!--more-->

## 背景

自己当前的博客是采用`GitHub Pages`+`GoDaddy`实现，已经使用了七八年，在此过程中除了感觉`GoDaddy`的域名续费有点小贵之外，没有遇到其它问题。

不过每年接近200元的域名费用，只用来托管自己质量和数量都不高的个人博客实在有些浪费，能否最大限制的挖掘其价值呢？

经过一番调研后最终决定采用`GitHub Pages`+`GoDaddy`子域名来托管多个项目供自己与展示使用。

## 多个GitHub Pages

在`GitHub`的官网中有关于`GitHub Pages`类型和使用限制的详细说明[^1]:

* 基于用户，访问方式为`http(s)://<username>.github.io`
* 基于组织，访问方式为`http(s)://<organization>.github.io`
* 基于项目，访问方式为`http(s)://<username>.github.io/project`

很显然，在基于用户的方式时只能创建一个`GitHub Pages`站点，虽然单个用户可以创建多个组织，但是`GitHub`官方限制同一个账户只能创建一个基于组织的`GitHub Pages`站点，且不能与基于用户方式的同时存在。

故此要想实现创建多个`GitHub Pages`站点只能采用基于项目的方式，相关操作步骤如下：

1. 进入个人的`GitHub`主页，切换到`Repositories`菜单下，点击右上角的`New`菜单创建一个新的仓库。需要注意的是此处选择的为`Repositories`而不是`Projects`，这是由于`GitHub`中的`Repositories`主要是用来管理代码，而`Projects`则是对项目进行管理，侧重非代码。

   !["切换菜单开始创建仓库"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/github-switch-to-create-new-repository.png "切换菜单开始创建仓库") 

2. 在出现的界面中输入一个不冲突的仓库名称，之后点击下方的`Create repository`按钮进行创建

   !["github创建仓库"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/create-github-repository.png "github创建仓库") 

3. 创建完毕后进入该项目，依次点击`Settings`->`Pages`菜单，出现如下页面，选择对应的分支和根目录，之后点击`Save`按钮，即可完成初步配置

   !["开启GitHub Pages"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/enable-github-pages.png "开启GitHub Pages") 

4. 将该仓库`clone`到本地，在根目录添加一个名为`index.html`的文件，内容如下，之后提交并推送到仓库

   ```html
   <!DOCTYPE html>
   <html>
   <body>
   <b>Book Test</b> 
   <div id="time"></div>
   </body>
   <script type="text/javascript">
   const date = new Date();
   
   const year = date.getFullYear();
   const month = String(date.getMonth() + 1).padStart(2, '0'); // Month is 0-based
   const day = String(date.getUTCDate()).padStart(2, '0');
   
   const hour = String(date.getHours()).padStart(2, '0');
   const minute = String(date.getMinutes()).padStart(2, '0');
   const second = String(date.getSeconds()).padStart(2, '0');
           
   const strDate = year + "-" + month + "-" + day + " " + hour + ":" + minute + ":" + second;
   
   document.getElementById("time").innerHTML = strDate;
   </script>
   </html>
   ```

5. 之后在浏览器中输入`https://lucumt.github.io/book`或`http://lucumt.github.io/book`，出现类似如下界面，表示基于项目的`GitHub Pages`站点创建成功。

   !["GitHub Pages测试"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/github-pages-test.png "GitHub Pages测试") 

   可以看出虽然测试正常，但上述访问方式需要将项目名包含进去，不太美观也不便于后续的统计扩展等操作，接下来需要配置`GoDaddy`有针对性的设置子域名。

## 创建与配置子域名

在`GoDaddy`的官网上有关于子域名的相关说明[^2]：

* 一个域名最多支持500个子域名
* 子域名可以嵌套，如在子域名`book.lucumt.info`下面创建嵌套的子域名`it.book.lucumt.info`
* 非嵌套子域名最长可以有255个字符，嵌套子域名则不能超过63个

从上可知我们完全可在`GoDaddy`上给自己购买的域名，添加多个子域名，相关操作步骤如下：

1. 使用个人账号登录`GoDaddy`主页，在右上角点击`我的产品`，出现类似如下界面，在对应域名后面点击`DNS`链接

   !["GoDaddy域名列表"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/godaddy-product-list-page.png "GoDaddy域名列表") 

2. 在出现的界面中点击`DNS记录`菜单下的`添加新纪录`按钮

   !["GoDaddy添加新记录按钮"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/godaddy-add-records-button.png "GoDaddy添加新记录按钮") 

3. `GoDaddy`中添加域名记录支持两种类型：`A记录`与`CNAME`，其中`A记录`指向具体的IP地址，`CNAME`指向其它的域名，本例中我们希望用来替代`lucumt.github.io/book`，故需要选择`CNAME`。

   仿照下述界面，填入相关信息并点击`保存`按钮，至此`GoDaddy`中的相关配置操作完毕，`GoDaddy`官网说需要1-48小时生效，但个人实际测试发现操作完毕后一分钟即能生效

   !["GoDaddy基于CNAME添加记录"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/godaddy-add-cname-record.png "GoDaddy基于CNAME添加记录") 

4. 在对应的`GitHub`项目中创建一个名为`CNAME`的文件，并**只写入`book.lucumt.info`**，然后提交推送到`GitHub`

5. 提交完毕之后待其默认构建完毕后，在`GitHub`中进入对应项目，依次点击`Settings`->`Pages`，可以看见`DNS`正在校验我们设置的二级域名，此过程可能需要3-15分钟左右才能检查完毕，在此期间无法开启`HTTPS`支持

   !["GitHub Pages中DNS检查域名"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/github-pages-dns-check-in-progress.png "GitHub Pages中DNS检查域名") 

6. 根据不同检查完毕后的输出类似如下，此时虽然开启`HTTPS`选项仍不可用，但可用`HTTP`协议测试

   !["GitHub Pages中DNS检查域名完毕"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/github-pages-dns-check-finished.png "GitHub Pages中DNS检查域名完毕") 

7. 在浏览器中输入`http://book.lucumt.info`会出现类似如下界面，表示我们的子域名绑定操作基本成功

   !["基于HTTP测试GoDaddy域名"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/github-pages-godaddy-http-test.png "基于HTTP测试GoDaddy域名") 

8. 若在步骤6中的开启`HTTPS`按钮一直不可用，可在浏览器中刷新该页面，之后出现的界面即可让我们选中开启`HTTPS`

   !["基于HTTP测试GoDaddy域名"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/github-pages-enable-https.png "基于HTTP测试GoDaddy域名") 

9. 之后在浏览器中输入`https://book.lucumt.info`，结果类似如下，可发现`HTTPS`支持也生效。

   至此，整个操作过程完毕！

   !["基于HTTPS测试GoDaddy域名"](/blog_img/github/create-multiple-github-pages-and-config-godaddy-subdomain/github-pages-godaddy-https-test.png "基于HTTPS测试GoDaddy域名") 

[^1]: [About GitHub Pages - GitHub Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#types-of-github-pages-sites)
[^2]: [什麼是子網域？ | 網域 - GoDaddy 說明 HK](https://hk.godaddy.com/help/what-is-a-subdomain-296)