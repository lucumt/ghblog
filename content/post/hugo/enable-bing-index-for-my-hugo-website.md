---
title: "记录如何解决个人博客没有被Bing检索收录的问题"
date: 2024-10-07T21:24:46+08:00
lastmod: 2024-10-07T21:24:46+08:00
draft: true
keywords: ["Hugo","blog","Google index","Bing Index","检索","收录"]
description: "基于个人经验，简要介绍通过给Bing客户发送邮件来解决基于Hugo的博客没有被Bing检索收录的问题"
tags: ["Hugo"]
categories: ["个人博客"]
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

自己的`Hugo`博客一直以来都无法被[**Bing**](https://www.bing.com/)收录检索，本文简要记录自己通过改进[**Hugo**](https://gohugo.io/)博客内容以及给`Bing`多次发送技术支持邮件来解决这一问题的过程。

<!--more-->

## 背景

自己的`Hugo`博客是基于[**GitHub Pages**](https://pages.github.com/)托管，在最开始几年自己没有花费太多时间打理，一直处于放养状态，导致个人博客对于除了`Google`之外的搜索引擎检索收录的效果一直不太好，基本近似于无。

最近几年由于工作变化以及其它方面的原因，个人开始花更多的时间经营博客，**主要目的是将自己认为有价值的知识点记录下来顺便给其他人提供一丢丢的参考**，除了提交更多的博文之外，也包括对博客样式的改进以及对搜索引擎的增强。

博客样式改进自己前面已经写过不少文章，但对于搜索引擎的支持一直以来都只有`Google`能做到全部收录，但由于众所周知的原因其在国内的访问受到限制，而国内常用的`Baidu`和`Bing`对于个人博客的收录却一直很差劲。考虑到`GitHub Pages`屏蔽了`Baidu`爬虫，同时其检索效果一般，作为一个自我定位为技术型的博客网站，自己把主要的关注点放在了如何实现`Bing`的检索与收录。

## 问题

在`Google`中输入`site:lucumt.info`时的效果如下，可以看到个人博客的内容基本上全部被检索收录了。

![Google检索收录效果](/blog_img/hugo/enable-bing-index-for-my-hugo-website/google_site_result.png "Google检索收录效果")

而在`Bing`中输入同样的指令时，结果如下，一个页面都没有收录！

![Bing无法收录](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing_site_result_not_working.png "Bing无法收录")

在`Baidu`中输入同样的指令时，结果如下，虽然有数据但聊胜于无，but who cares! 我只想让`Bing`能收录即可。

![Baidu收录结果](/blog_img/hugo/enable-bing-index-for-my-hugo-website/baidu_site_result.png "Baidu收录结果")

## 分析&解决

### 邮件咨询

登录[**Bing Webmaster**](https://www.bing.com/webmasters)后查看对应的数据，一直提示`Crawled but not indexed`,然后

### 页面修改

基于反馈结果，主要有如下两个地方的修改

1.在`Hugo`博客的相关页面头部修改`title`和`description`属性，确保它们不能太短或太长，对于`title`属性其长度建议不能超过65个字符，对于`description`属性其长度建议在`25-160`个字符之间

```yaml
title: "给Hugo中的Even主题添加搜索功能"
date: 2023-11-30T10:35:48+08:00
lastmod: 2023-11-30T10:35:48+08:00
draft: false
keywords: ["hugo","search"]
description: "参考网络文档给Hugo中的Even主题添加搜索功能，以方便检索博客的内容"
tags: ["hugo"]
categories: ["个人博客"]
author: "Rosen Lu"
```

2.若页面中需要显示目录层级时，要确保从`h2`而非`h1`开始。基于SEO准则，同一个页面建议只有一个`h1`标签，虽然`Google`目前支持同一个页面多个`h1`标签，但`Bing`等搜索引擎尚不支持。个人使用的`Even`主题默认将博客标题设置为`h1`级别，处于兼容性考虑，需要将文章段落标题从`h2`开始。

### 测试网站

按照前述说明对网站内容进行调整后，等了一周左右发现`Bing`还没有收录检测我的网站，通过`Bing Webmaster`进行查看时

### 正常收录

将博客内容修改完毕之后，

![Bing正常收录](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing_site_result_working.png "Bing正常收录")

同时在`Bing Webmaster`中也能查看正常的数据

![BingWebmaster显示正常检索](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-webmaster-indexed-result.png "BingWebmaster显示正常检索")

## 总结

本以为添加`Bing`检索收录会很简单，但从着手实施到最终有结果，整个过程断断续续的持续了4个月，有必要将其总结记录下来给有需要的人参考：

1. 首先一定要阅读`Bing`官方提供的指导说明，确保个人博客中的内容没有违反其相关约束
2. 通过`Bing Webmaster`工具对网站整体进行检测，同时可选取几个典型的文章通过工具对文章内容样式和质量进行检查
3. 通常若前2个步骤执行完毕后即可被`Bing`正常检索收录，若还不生效则有可能是网站由于各种原因被`Bing`加入黑名单了此时需要给其发邮件寻求官方支持
4. 基于网络上其他人的相关经历，给`Bing`发邮件后通常而言都能解决问题，若邮件回复还是说网站质量有问题，可采用当前域名的子域名创建一个类似的简单测试网站排查下是否为域名本身的问题。若采用子域名测试依然有问题，可等待一两周后给`Bing`的技术支持人员发一封**新的邮件**反馈该问题，或许`Bing`官方换一个技术支持人员(本人即通过此种方式实现被`Bing`收录)。此处要注意是最好重新发一封邮件不要在原有的邮件上直接回复，否则会遇到类似我这种技术支持人员踢皮球的回复，根本解决不了任何问题。
5. 由于国内主流的搜索引擎依旧是`Baidu`，对于托管在`GitHub Pages`上的网站若想被其收录，要么切换到能被`Baidu`爬虫访问到的服务器上(如自建服务器、购买VPS、国内的腾讯code托管服务等)上，或在博客园、掘金、公众号等平台上对原有的平台进行引流。

