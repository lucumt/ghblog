---
title: "利用GitHub Action实现Hugo博客在GitHub Pages自动部署"
date: 2022-09-03T18:35:57+08:00
lastmod: 2022-09-03T18:35:57+08:00
draft: false
keywords: ["hugo","theme","even"]
description: "简要叙述利用GitHub Action实现Hugo博客在GitHub Pages自动部署"
tags: ["hugo","github-pages","go"]
categories: ["个人博客"]
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

sequenceDiagrams: 
  enable: false
  options: ""
---

作为一名IT民工，善于利用各种工具提升工作效率才算合格，本文简单记录自己如何利用[GitHub Actions](https://github.com/features/actions)实现个人[Hugo](https://gohugo.io/)博客在[GitHub Pages](https://pages.github.com/)中的自动化部署。 

<!--more-->

# 传统方式

自己的个人博客创建于2016年，在这期间自己一直基于如下方式创建并部署更新博客：

1. 利用`hugo`命令创建对应的博客markdown文件

   `hugo new post/hugo/using-github-action-to-auto-build-deploy.md`

2. 利用下述命令开启`hugo`博客的动态监听展示，并进行编写

   `hugo server -w -D`

3. 博客内容编写完成后，利用下述命令将其切换到实际部署环境

   `hugo server --baseUrl="https://lucumt.info/" --watch=false --appendPort=false --renderToDisk --environment production`

4. 执行下述命令提交到`master`分支

   * `git add -A`
   * `git commit -a -m "xxxx"`
   * `git push origin master`

5. 利用下述命令将`public`目录中的内容从`master` 分支同步到`gh-pages`分支

   `git subtree push --prefix=public git@github.com:lucumt/ghblog.git gh-pages`

上述过程中的1,2,4阶段是编写博客的必经阶段，而3,5阶段其实没太多必要，完全可以用工具自动化实现。作为IT从业者，我们需要尽可能的减少不必要的操作。

# 改进方式 

结合网络上的相关资料，自己把实现方案定在了`GitHub Actions`和[Travis CI](https://www.travis-ci.com/)二者之一，考虑到`GitHub`中已经内置了`GitHub Actions` ，最终解决采用其作为实现方案。

在`Hugo`的官方文档[Build Hugo With GitHub Action](https://gohugo.io/hosting-and-deployment/hosting-on-github/#build-hugo-with-github-action)中也推荐采用`GitHub Actions`作为持续集成部署方案，并提供了相应的流水线配置代码:

```yaml
name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

该配置代码已经很完善，个人根据实际情况对其做了如下修改：

1. 在Build阶段，将`hugo`命令改为适合个人环境的`hugo -b "https://lucumt.info/" -e "production"`
2. 在个人`GitHub`中设置`github_token`

其中关于`github_token`的配置可按如下步骤配置：

1. 在个人`GitHub`页面，依次点击`Settings`->`Developer settings`->`Personal access tokens`进入如下页面：

   ![设置personal access token](/blog_img/hugo/using-github-action-to-auto-build-deploy/generate-new-token.png "设置personal access token")  

2. 点击`Generate new token`出现如下界面，在Note中输入名称，在Select scopes选择`workflow`

   ![设置token信息](/blog_img/hugo/using-github-action-to-auto-build-deploy/set-personal-access-token.png "设置token信息")  

3. 将生成的token复制出来为后续创建`secret`做准备，注意必须及时复制，一旦离开此页面后续就无法查看其值，只能重新创建新token：

   ![复制生成的token](/blog_img/hugo/using-github-action-to-auto-build-deploy/generate-new-token-result.png "复制生成的token")  

4. 进入对应的`GitHub`项目下，依次点击`Settings`->`Secrets`->`Actions`进入添加`Action secrets`的界面，点击`New repository secret`按钮

   ![创建action secret token](/blog_img/hugo/using-github-action-to-auto-build-deploy/generate-action-secrets.png "创建action secret token")  

5. 在出现的界面中`name`部分输入我们设置的值，`Secret`部分输入步骤3中记录的token值，然后点击`Add secret`按钮

   ![设置action secret](/blog_img/hugo/using-github-action-to-auto-build-deploy/set-action-secrets.png "设置action secret")  

   需要注意的是`name`的值不能以`GITHUB_`开头，否则创建会出错

   ![设置action secret失败](/blog_img/hugo/using-github-action-to-auto-build-deploy/generate-action-secrets-name-violation.png "设置action secret失败")  

6. 在流水线中将`github_token`值设置为步骤5中`secret`的名称，类似`${{ secrets.GH_PAGE_ACTION_TOKEN }}s`，至此`github_token`设置过程完毕。

配置后完整的流水线代码如下：

```yaml
name: pages-auto-build-deploy
on:
  # workflow_dispatch: 
  push:
    branches:
      - master

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.100.2'
          extended: true

      - name: Build Hugo
        run: hugo -b "https://lucumt.info/" -e "production"

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GH_PAGE_ACTION_TOKEN }}
          publish_dir: ./public
          commit_message: ${{ github.event.head_commit.message }}
```

将该`yaml`文件放到对应`GitHub`项目下的`.github/workflows`目录下即完成全部配置。

当执行`git push origin master`后，`GitHub Actions`会开启自动构建部署，运行结果如下，至此整个设置过程完毕！

![GitHub Actions执行结果](/blog_img/hugo/using-github-action-to-auto-build-deploy/hugo-automatic-build-result.png "GitHub Actions执行结果")  

参考文档:

1. https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html
2. https://www.pseudoyu.com/zh/2022/05/29/deploy_your_blog_using_hugo_and_github_action/

