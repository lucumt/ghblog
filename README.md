![](https://img.shields.io/github/forks/lucumt/ghblog)
![](https://img.shields.io/github/stars/lucumt/ghblog)
![](https://img.shields.io/github/license/lucumt/ghblog)
![](https://img.shields.io/github/actions/workflow/status/lucumt/ghblog/pages-automatic-deploy.yml)
![GitHub language count](https://img.shields.io/github/languages/count/lucumt/ghblog)
![GitHub top language](https://img.shields.io/github/languages/top/lucumt/ghblog)
<br/>
![GitHub last commit (by committer)](https://img.shields.io/github/last-commit/lucumt/ghblog)
![GitHub commit activity (branch)](https://img.shields.io/github/commit-activity/m/lucumt/ghblog)
![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/lucumt/ghblog)
![GitHub repo size](https://img.shields.io/github/repo-size/lucumt/ghblog)
<br/>
![GitHub search hit counter](https://img.shields.io/github/search/lucumt/ghblog/Hugo)
![GitHub search hit counter](https://img.shields.io/github/search/lucumt/ghblog/Even)
![GitHub search hit counter](https://img.shields.io/github/search/lucumt/ghblog/Java)
![GitHub search hit counter](https://img.shields.io/github/search/lucumt/ghblog/Spring)
![GitHub search hit counter](https://img.shields.io/github/search/lucumt/ghblog/MySQL)
![GitHub search hit counter](https://img.shields.io/github/search/lucumt/ghblog/Golang)


# 我的个人博客
个人博客的源代码和相关说明。

## 技术实现
- 访问地址: **[https://lucumt.info](https://lucumt.info)**
- 博客引擎: **[Hugo](https://gohugo.io/)**（`hugo v0.126.1`）
- 域名托管: **[GoDaddy](https://www.godaddy.com)**
- 博客托管: **[Github Pages](https://pages.github.com/)**  

其中[Hugo](https://gohugo.io/)博客引擎的样式采用[hugo-theme-even](https://github.com/olOwOlo/hugo-theme-even),博客搭建请参见[利用Github Pages和基于Go的Hugo搭建个人博客](http://lucumt.info/post/hugo/create-website-with-hugo/) 

## 博客截图
![个人博客截图](static/blog_img/lucumt.info.png)  

# 其它

## 参考说明

* [利用Github Pages和基于Go的Hugo搭建个人博客](https://lucumt.info/post/hugo/create-website-with-hugo/)
* [在Hugo生成的博客中动态的修改样式](https://lucumt.info/post/hugo/change-hugo-style-in-even-theme/)
* [将基于Github Pages的自定义域名博客迁移到HTTPS](https://lucumt.info/post/hugo/migrate-github-blog-from-http-to-https/)

## 引用说明

请在Fork或Clone后将 **config.toml**中的 **gaid** 和 **baiduanalysis** 修改为你自己的相关账号，或者直接将这两个账户置为空。  

```toml
title = "飞狐的部落格"

logoTitle = "Rosen's World"
keywords = ["Hugo", "theme","even"]
description = "飞狐的个人博客"
  
baiduPush = true        # baidu push                 
baiduAnalytics = "cabc0a71f63da092412d82d1aefe7d1c"      
baiduVerification = ""   # Baidu Verification
googleVerification = ""  # Google Verification

googleAnalytics = "UA-75123653-1"  
```

## License

**[MIT License](https://en.wikipedia.org/wiki/MIT_License)**
