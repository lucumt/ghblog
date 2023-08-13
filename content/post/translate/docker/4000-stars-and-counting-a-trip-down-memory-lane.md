---
title: "[译]4000个点赞并且在不断增加，回忆之旅"
date: 2023-08-13T17:49:14+08:00
lastmod: 2023-08-13T17:49:14+08:00
draft: false
keywords: ["docker","docker-compose","prometheus"]
description: "翻译docker早期用户回忆Docker Compose是如何诞生的"
tags: ["docker"]
categories: ["翻译"]
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

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

<style>
    .content img {
      margin-left: auto;
      margin-right: auto;
      display: block !important;
    }
</style>



{{% admonition note "说明" false %}}

在阅读阮一峰老师的[**科技爱好者周刊（第 266 期）**](https://www.ruanyifeng.com/blog/2023/08/weekly-issue-266.html)时，发现其中一篇关于[**Docker Compose**](https://docs.docker.com/compose/)的文章[**4'000 Stars and counting, a trip down memory lane**](https://brianchristner.io/4000-stars-and-counting-a-trip-down-memory-lane/)很有意思，基于自己粗浅的英文翻译一下。

{{% /admonition %}}

<!--more-->

---

正文：

> 回忆之旅，`Docker Compose`以及使用`Docker`和`Prometheus`进行监控的历史，在这个过程中，我发现了一个很棒的社区。

![博客默认顶部图片](/blog_img/translate/4000-stars-and-counting-a-trip-down-memory-lane/photo-above-content.jpg "博客默认顶部图片")

<figcaption>

由<a href="https://unsplash.com/de/@clarissemeyer?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Clarisse Meyer</a>/<a href="https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit"> Unsplash</a>拍摄

</figcaption>

在2014年我很幸运成为第一批踏上`Docker`之旅并尝试将监控堆栈容器化的人之一，从年初开始了解`Docker`，我花了几个月时间弄清楚它是如何工作的，同时在工程师间的DM's[^1]中修复问题并提出了大量问题，当时只有少数人在聊天，社区很小以至于大家互相之间都认识。



作为一名监控和数据极客，在2014年我负责为我们新的云创业(`Cloud venture`)构建适应性更强的监控解决方案。最开始，我试图在`Docker`中构建当前的监控解决方案。就像容器友好之前的`Nagios`、`Centreon`、`Piwik` 分析以及许多其它工具一样，这些工具并没有真正解决我的问题。我通过`Grafana`偶然发现了`InfluxD`，但很快意识到它不符合我们的使用场景。



经过对不同工具的多次尝试和错误后，我偶然间发现Soundcloud基础设施博客以及他们如何基于微服务架构构建用`Go`编写的时间序列监控工具，在当时，这似乎太过超前和牵强，甚至看起来不真实。如果我没记错的话，就是这篇博客文章，它将`Prometheus`推向了公众：[**Prometheus: Monitoring at SoundCloud**](https://developers.soundcloud.com/blog/prometheus-monitoring-at-soundcloud)。



 **首先，我们要容器化一切。**

回到2014年，相信与否，这是一个没有编排器的时代，大多数人将一堆单个`Docker`运行命令与`bash`脚本串在一起，并尝试使用`bash`脚本“编排”容器。



幸运的是，在2015 年，一家名为Orchard的公司正忙于构建名为`Fig`的第一个`Docker`容器编排器。`Fig`是一个绝对令人惊奇的工具，你可以用`YAML`来编写一些奇怪的代码，这是我之前从未见过的。基本上，将所有 `Docker`运行命令组合到一个文件中，从而允许我们将微服务构建到一个文件中。

![Mind Blown](/blog_img/translate/4000-stars-and-counting-a-trip-down-memory-lane/wow-amazed.gif "Mind Blown")

<figcaption>用单个文件运行<code>Docker</code>命令==令人震惊</figcaption>

在我开始使用Fig并加快步伐后不久，`Docker`宣布[**收购Fig的母公司：Orchad**](https://www.informationweek.com/infrastructure-as-a-service/docker-acquires-devops-flavor-with-fig?ref=brianchristner.io#)，我们还可以在`Docker Compose`版本历史记录中看到`Fig`于2014年10月加入`Docker`时的精彩历史快照。



几个月之后，`Docker`在2015年2月宣布将[**`Fig`更名为`Docker Compose`**](https://docs.docker.com/compose/release-notes/?ref=brianchristner.io#110)，它是容器发展史上我永远不会忘掉的一个里程碑。从那之后，我开始研究如何将我所有我喜欢的监控工具整合到一个文件中，我开始使用`Prometheus`、`Grafana`、`Node Exporters`和`Google cAdvisor`。我花了几周的时间才弄清楚`Docker Compose`内部的网络，因为还处于早期阶段，我们今天认为理所当然的很多东西在当时还不可用。让容器通过多个服务相互通信、配置、安装存储等是一项艰巨的任务，但最后确实生效了。



下图展示了我使用`Prometheus`和`Grafana`创建的第一个仪表板，它并不美观，但是嘿，它们是基于`Docker Compose`编排与协同工作的，同时还能监视容器和基础设施。

![Dashboard_Prometheus](/blog_img/translate/4000-stars-and-counting-a-trip-down-memory-lane/dashboard_drometheus.png "Dashboard_Prometheus")

<figcaption>基于<code>Prometheus</code>和<code>Grafana</code>的监控面板初始版本</figcaption>



在它启动并成功运行之后，我在博客上写了几篇关于它的文章，与此同时获得了来自`Google`、`Docker`和整个社区的广泛关注，`Google`还主动联系并要求我在他们位于瑞士苏黎世的总部展示我正在构建的东西。



我不确定我是否为第一个使用`Docker Compose`创建完整监控堆栈的人，但我绝对是第一个创建详细手册、简单的一键部署指南，参加`Docker`监控研讨会培训并100人如何开始使用`Docker`和监控的人。很快我成为第一批`Docker Captains`之一，写书、参加采访，并很早就成为`Docker`社区不可或缺的一部分。



对我来说始终很重要的一点是：Docker Prometheus项目的目标是帮助他人并回馈社区，开源不仅对于代码编写非常重要，而且对于支持、教导和指导他人也非常重要，我相信这个项目帮助启动了其它的一些项目和组织，以继续构建令人惊叹的作品。



我知道的第2件事情：Docker Prometheus GitHub项目有1k点赞，然后是2k，现在是4k，`GitHub`项目的点赞数在早期有特殊的含义，一些公司使用它们作为向投资机构推销的指标，而现在它只是一个虚荣指标，但我仍然喜欢不时的对自己的项目进行检查和反思。



这是一段尚未完成的奇妙旅程，我鼓励每个人尽其所能帮助并回馈开源项目，从编写代码到修复文档，每一个贡献都是有益的。



感谢Solomon Hykes、Scott Johnston、Bret Fisher、Laura Tacho、Julien Bisconti，所有`Docker Captains`以及`Docker`和整个`Docker`社区的每个人。

<–翻译结束!–>

[^1]: 注：全称为Direct Message，即私有消息，一对一聊天，`Twitter`中的常用语法