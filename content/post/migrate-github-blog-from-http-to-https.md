+++
author = "飞狐"
categories = ["个人博客"]
tags = ["Github Pages","HTTPS"]
date = "2017-12-24T10:28:03+08:00"
description = "将基于Github Pages的自定义域名博客迁移到HTTPS"
keywords = ["Github Pages","HTTPS"]
title = "将基于Github Pages的自定义域名博客迁移到HTTPS"

+++

越来越多的网站和个人博客都变成 **[HTTPS](https://en.wikipedia.org/wiki/HTTPS "https://en.wikipedia.org/wiki/HTTPS")** ，而自己的博客一直都是用的是 **[HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol "https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol")** 协议，作为一个具有强迫症的人，每次用 **[Chrome](https://www.google.cn/chrome/browser/desktop/index.html "https://www.google.cn/chrome/browser/desktop/index.html")** 浏览器打开个人博客时看见浏览器地址栏显示的!["Chrome HTTP标识"](/blog_img/migrate-github-blog-from-http-to-https/http_icon.png "Chrome HTTP标识") 都感觉很不舒服。趁着前段时间不太忙，将个人博客从 *HTTP* 迁移到了 *HTTPS* ，先记录下。

<!--more-->
<br/>
一开始我想直接通过在 **[GoDaddy](https://www.godaddy.com/)** 上直接购买 *HTTPS* 服务来实现，去官网查看后发现费用太高，一年大约100美刀，果断弃之。 Google后发现很多人都用 **[Cloudflare](https://www.cloudflare.com/)** 通过转发请求来实现 *HTTPS* 访问，操作起来也很快，自己便也采用 *Cloudflare* 实现， 本文主要是基于 *Cloudflare* 的实现说明。

## 操作过程

### 给博客添加自定义域名

本人使用的是 *GoDaddy* 来设置自定义域名，具体操作请参见 **[利用GoDaddy配置自定义域名](https://lucumt.info/posts/create-website-with-hugo/#%E5%88%A9%E7%94%A8godaddy%E9%85%8D%E7%BD%AE%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9F%9F%E5%90%8D:ef8b9e40461ea61e62e36d1aa4c54d14)**， 核心的操作就是给 **CNAME** 文件添加 *Github Pages* 给出的两条**A**记录IP地址，此处不再详述。

### 利用Cloudflare修改DNS服务器

1. 打开 *Cloudflare官网* 注册一个 *Cloudflare* 账户。注册成功之后，点击页面右上角的 *add site* 链接，添加一个网站，在下图输入框中输入自己的域名，点击 Begin Scan 按钮开始扫描。  
!["在Cloudflare中扫描站点"](/blog_img/migrate-github-blog-from-http-to-https/add_site_in_cloudflare.png "在Cloudflare中扫描站点")  
2. 扫描完毕后点击 *Contiue Setup* ，在类似如下图所示的界面中选择 *Free Website* ，然后点击页面底部的 *Continue* 按钮。  
!["选择Cloudflare免费站点"](/blog_img/migrate-github-blog-from-http-to-https/cloudflare_free_website.png "选择Cloudflare免费站点")  
3. 在下图所示的 *Cloudflare Nameservers* 说明界面中根据要求来修改自定义域名的DNS服务器。
!["Cloudflare扫描结果"](/blog_img/migrate-github-blog-from-http-to-https/cloudflare_nameservers.png "Cloudflare扫描结果")  
4. 登录 *GoDaddy* ，打开响应域名的 *Manage DNS* 界面，将 *Nameservers* 从 *Default* 修改为 *Custom* 然后添加前一个步骤中的两个值分别加上并点击保存。  
!["Cloudflare扫描结果"](/blog_img/migrate-github-blog-from-http-to-https/modify_dns_server_in_godaddy.png "Cloudflare扫描结果")  
5. 回到 *Cloudflare* 网站，点击 *Overview* 按钮，查看域名的状态是否为如下所示的 *Active*，若是则表示DNS服务器修改成功，若不是 *Active* 请等待几分钟。  
!["Cloudflare中配置DNS成功"](/blog_img/migrate-github-blog-from-http-to-https/cloudflare_website_actived.png "Cloudflare中配置DNS成功")  

### Cloudflare中开启HTTPS设置

1. 在 *Cloudflare* 网站上点击顶部的 *Crypto* 按钮，将状态修改为 *Full*。  
!["Cloudflare中SSL设置"](/blog_img/migrate-github-blog-from-http-to-https/cloudflare_ssl_setting.png "Cloudflare中SSL设置")  
2. 在顶部切换到 *Page Rules* 界面，点击 *Create Page Rule* ，添加规则`http://lucumt.info/*` 并选择 *Always Use HTTPS* 来强制该域名下的所有请求都是用HTTPS实现，然后点击 *Save and Deploy* 来部署该规则，注意此处的规则是 *HTTP* 而不是 *HTTPS* 。  
!["Cloudflare中Page Rule设置"](/blog_img/migrate-github-blog-from-http-to-https/cloudflare_page_rule_setting.png "Cloudflare中Page Rule设置")  
执行完这一步后理论上通过 *HTTPS* 可以正常的访问个人博客，但还需对博客源码做一些修改。

### 将代码中所有HTTP修改为HTTPS
在执行完前面的步骤后，在浏览器中用 *HTTPS* 访问个人博客时，可能看到的还是!["Chrome HTTP标识"](/blog_img/migrate-github-blog-from-http-to-https/http_icon.png "Chrome HTTP标识")而不是自己期望中的小绿锁，同时浏览器控制台可能会出现类似 *Mixed Content,The page at ...,The request has been blocked,the content must be serverd over HTTPS* 的错误信息。其原因是由于某些页面中存在混合内容，即部分请求还是以 *HTTP* 方式实现的，如加载 *Javascript* 、 *CSS* 等，解决方法也很简单，**将所有的HTTP请求都改为HTTPS即可** 。

以我个人基于 **[Hugo](https://gohugo.io/)** 的博客为例，要进行如下几步操作：

1. 将个人 *Hugo* 源代码中所有的 *HTTP* 请求都修改为 *HTTP*,包括页面中的直接请求和 *Javascript* 、 *CSS* 等文件中的间接请求。 
2. 利用 `hugo server -D --baseUrl="https://lucumt.info" --appendPort=false` 重新生成基于 *HTTPS* 的源文件页面。
3. 将生成的博客源代码重新上传到 *Github* 仓库中，经过1分钟左右以 *HTTPS* 的方式在浏览器中打开个人博客，期待中的小绿锁入愿出现，世界终于和谐了！  
!["通过HTTPS打开个人博客"](/blog_img/migrate-github-blog-from-http-to-https/open-blog-website-via-https.png "通过HTTPS打开个人博客")  


## 注意事项
通过 *Cloudflare* 虽然可以快速的将自己的个人博客迁移到 *HTTPS* ，但基于以下两个方面的原因，在条件许可的情况下还是应该使用其他方式实现HTTPS:

- 免费计划下的 *Cloudflare* 实际上相当于一个中介，我们的访问请求先被 *Cloudflare* 代理实现 *HTTPS* 接收到然后将其转发给原始的服务器（如 *Github Pages* 服务器）。虽然浏览器与 *Cloudflare* 之间的通信是 *HTTPS* 加密，但是 *Cloudflare* 与实际服务器之间的通信不一定是加密的，存在被挟持和篡改的可能。 *Cloudflare* 之前也被爆出过 **[安全漏洞和敏感数据泄露](https://thehackernews.com/2017/02/cloudflare-vulnerability.html "Serious Bug Exposes Sensitive Data From Millions Sites Sitting Behind CloudFlare")**，故商业网站通常不用 *Cloudflare* 免费计划，但个人博客出于增加搜索引擎收录和省钱等原因，可以使用 *Cloudflare* 免费计划。   
  !["Cloudflare代理实现HTTPS"](/blog_img/migrate-github-blog-from-http-to-https/cloudflare_ssl_website.png "Cloudflare代理实现HTTPS")
- 由于经过 *Cloudflare* 这层代码，访问速度肯定没有直接访问原始服务器那么快，对于响应速度要求高的用户不合适。
- 由于众所周知的原因，在天朝 *Cloudflare* 访问速度比国内慢，而且指不定哪天就被ban了。

参考:

- [https://bakumon.me/blog/p/github-pages-https-ssl.html](https://bakumon.me/blog/p/github-pages-https-ssl.html)
- [https://help.github.com/articles/securing-your-github-pages-site-with-https/](https://help.github.com/articles/securing-your-github-pages-site-with-https/)
