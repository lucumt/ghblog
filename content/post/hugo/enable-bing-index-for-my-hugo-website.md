---
title: "记录如何解决Hugo博客没有被Bing检索收录的问题"
date: 2024-10-07T21:24:46+08:00
lastmod: 2024-10-07T21:24:46+08:00
draft: false
keywords: ["Hugo","blog","Google index","Bing Index","检索","收录"]
description: "基于个人经验，简要介绍通过给Bing客户发送邮件来解决基于Hugo的博客没有被Bing检索收录的问题"
tags: ["Hugo"]
categories: ["个人博客"]
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
moreMeta: true

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

{{% admonition info "简单版解决方案总结" false %}}

确保自己的网站在`Bing Webmaster`检测时不出现错误以及尝试给`Bing`不同的技术支持人员发送邮件反馈问题。

{{% /admonition %}}

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

### 网站检查

登录[**Bing Webmaster**](https://www.bing.com/webmasters)后查看对应的数据，显示的效果类似下图，虽然有很多页面，但大部分状态为`Excluded`,导致`Bing`一直无法检索收录。

![WebMaster显示页面没有被收录](/blog_img/hugo/enable-bing-index-for-my-hugo-website/webmaster_show_page_not_indexed.png "WebMaster显示页面没有被收录")

进入到`URL Inspection`菜单后选中若干个典型页面地址进行检测时，都是类似如下的提示，虽然从结果中知道相关页面违反了`Bing`的规则，但具体是违反了哪些规则却没有明确告诉我们，给排查问题增加了难度(貌似个人遇到的几个国外网站都是类似，只是说你违反了他们的规则，但是具体违反了哪方面死活不说)。

![网页地址无法被bing收录](/blog_img/hugo/enable-bing-index-for-my-hugo-website/url-can-not-being-indexed-by-bing.webp "网页地址无法被bing收录")

### 页面修改

根据[**Bing Webmaster Guidelines**](https://www.bing.com/webmasters/help/webmasters-guidelines-30fba23a)中的说明尤其是[**Things to Avoid**](https://www.bing.com/webmasters/help/webmasters-guidelines-30fba23a#avoid)中的相关规则，对自己的博客页面进行检查后，初步发现主要是如下3个地方不符合要求：

1. 部分文章的标题长度太长或太短，官方建议不能超过`65`个字符
2. 部分文档的描述信息太短，关键建议是描述信息长度在`25-160`个字符之间
3. 大部分页面的标题层级不合适都是从`h1`开始，基于SEO准则，同一个页面建议只有一个`h1`标签，虽然`Google`目前支持同一个页面多个`h1`标签，但`Bing`等搜索引擎尚不支持。

基于上述检查结果，对个人博客相关页面进行如下两个方面的修改：

1.在`Hugo`博客的相关页面头部修改`title`和`description`属性，确保其符合要求

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

2.个人使用的`Even`主题默认将博客标题设置为`h1`级别，若页面中需要显示目录层级时，出于兼容性考虑需要将文章段落标题从`h2`开始。

改完之后，网络上的资料说至少需要等待24小时，但是自己等待了一周之后结果依旧，还是不能被`Bing`收录！

采用前述的方式对个人博客中的页面进行`URL Inspection`后还是提示无法检索，这下自己不知道咋办了。

### 邮件咨询

`Bing`也是一个很成熟的搜索引擎，既然其说明我个人博客违反了相关规定，那肯定还是我自己的问题，但前面已经按照其要求进行了对应的更改，为啥还是有问题呢？

我首先想到的是找几个其它采用`Hugo`的类型网站进行对比，如`Even`主题作者的网站[olowolo.com](https://olowolo.com/)，经过对比之后并没有发现有啥特殊之处，但奇怪的是别人的能被收录自己的却不能。

没办法，只能尝试其它方法，通过[**Bing Webmaster Support**](https://www.bing.com/webmasters/support)给其发了一封邮件

![给Bing发送技术支持邮件](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-support-request-email-1.png "给Bing发送技术支持邮件")

第3天收到了一封机械式的回复，内容和我之前猜想的一样，就是告诉你有地方不符合要求，但具体是哪些地方却不告诉你。

![Bing技术支持人员初次回复](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-support-request-email-reply-1.png "Bing技术支持人员初次回复")

### 测试网站

为了进一步的确认是自己域名问题还是`Hugo`网站内容质量问题，我基于个人域名`lucumt.info`创建了两个子域名`tech.lucumt.info`和`book.lucumt.info`并创添加了相应的内容。作为对比，其中[tech.lucumt.info](https://tech.lucumt.info)是采用其它的`Hugo`框架生成的页面，[book.lucumt.info](https://book.lucumt.info)只有一个页面且采用了与`Hugo`无关的纯HTML编写。

基于`Bing`的相关规则，确保这2个网站没有违法规则，之后自己将它们也添加相关的配置，期望其能够被`Bing`收录，遗憾的是等待了半个月后，这2个子域名网址都没有被收录，即使是一个简单的单页面也是一样！

难道是我的域名问题？参考网络上其他人的解决方案，都建议给`Bing`发邮件解决，于是我在已有的邮件上回了一封邮件

![再次给Bing发送技术支持邮件](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-support-request-email-2.png "再次给Bing发送技术支持邮件")

不出所料，即使我在邮件中说明我已经仔细阅读了相关说明，还是毫无卵用的机械式回复

![Bing技术支持人员第2次回复](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-support-request-email-reply-2.png "Bing技术支持人员第2次回复")

后面我又在此基础上发邮件咨询了几次，但结果都类似，没有解决任何问题

![Bing邮件沟通列表](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-support-request-email-list.png "Bing邮件沟通列表")

感到很愤怒的我回复了下面一封很不礼貌的邮件，然后`Bing`技术支持人员再也没给我回复了~

![最后一次给Bing发送技术支持邮件](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-support-request-email-3.png "最后一次给Bing发送技术支持邮件")

发送邮件寻求帮助这条路看起来走不通！

### 再次邮件

尝试了包括邮件咨询，创建对比网站等方式，网站还是无法被收录，个人已经技穷，一度考虑是否要放弃`Bing`收录，毕竟`Google`能正常收录也还说的过去。

我这个人有时候有那么一点较真，为啥别人的网站都能被收录，就我的不能被收录呢，我试着通过[**Bing Webmaster Support**](https://www.bing.com/webmasters/support)给其发了一封新的邮件

![重新给Bing发送技术支持邮件](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-support-request-email-4.png "重新给Bing发送技术支持邮件")

不同于以往的机械式回复，这次其官方人员回复说我提供的3个网站中有1个可以被收录，另外2个不能被收录

![Bing技术支持人员新的回复](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-support-request-email-reply-3.png "Bing技术支持人员新的回复")

虽然问题没有彻底解决，但至少有一个网站能被收录也算是一个良好的开端！

### 问题解决

邮件中建议等待2-3周以便让其生效，但实际上我只等待了3天就发现[tech.lucumt.info](https://tech.lucumt.info)已经被`Bing`收录。

神奇的是，在`Bing`搜索栏中使用`site:lucumt.info`可发现其也被收录！

![Bing正常收录](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing_site_result_working.png "Bing正常收录")

同时在`Bing Webmaster`中也能查看正常的数据，看起来3个网站都已经被正常收录。

![BingWebmaster显示正常检索](/blog_img/hugo/enable-bing-index-for-my-hugo-website/bing-webmaster-indexed-result.png "BingWebmaster显示正常检索")

奇怪的是之前的邮件告诉我只有1个网站符合要求可以被收录，另外2个还是有问题，但为啥这会都好使了呢？

初步猜测是个人域名有问题导致整体被封锁，现在1个子域名被解锁后导致其余的也生效，虽然具体原因还不确定，但至少个人的问题初步解决了。

## 总结

本以为添加`Bing`检索收录会很简单，但从着手实施到最终有结果，整个过程断断续续的持续了4个月，有必要将其总结记录下来给有需要的人参考：

1. 首先一定要阅读`Bing`官方提供的指导说明，确保个人博客中的内容没有违反其相关约束
2. 通过`Bing Webmaster`工具对网站整体进行检测，同时可选取几个典型的文章通过工具对文章内容样式和质量进行检查
3. 通常若前2个步骤执行完毕后即可被`Bing`正常检索收录，若还不生效则有可能是网站由于各种原因被`Bing`加入黑名单了此时需要给其发邮件寻求官方支持
4. 基于网络上其他人的相关经历，给`Bing`发邮件后通常而言都能解决问题，若邮件回复还是说网站质量有问题，可采用当前域名的子域名创建一个类似的简单测试网站排查下是否为域名本身的问题。若采用子域名测试依然有问题，可等待一两周后给`Bing`的技术支持人员发一封**新的邮件**反馈该问题，或许`Bing`官方换一个技术支持人员(本人即通过此种方式实现被`Bing`收录)。此处要注意是最好重新发一封邮件不要在原有的邮件上直接回复，否则会遇到类似我这种技术支持人员踢皮球的回复，根本解决不了任何问题。
5. 由于国内主流的搜索引擎依旧是`Baidu`，对于托管在`GitHub Pages`上的网站若想被其收录，要么切换到能被`Baidu`爬虫访问到的服务器上(如自建服务器、购买VPS、国内的腾讯code托管服务等)上，或在博客园、掘金、公众号等平台上对原有的平台进行引流。
