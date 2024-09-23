---
title: "Embed Multiple Google Trends Charts Into Single Html Page"
date: 2024-06-14T11:07:55+08:00
lastmod: 2024-06-14T11:07:55+08:00
draft: false
keywords: ["Google Trends","Embed","Multiple"]
description: "A brief description of how to embed multiple Google Trends charts into a single html Page"
tags: ["html"]
categories: ["Web编程"]
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
centerImage: true
borderImage: false

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

<style type="text/css">
  #content {
     font-family:math;font-size:13pt;margin:1em 0;
  }

  .post .post-header .post-title,
  .post .post-content h1{
     font-family:revert;
  }
</style>


A brief descripton about how to embed multiple google trends chart into single html page to make them working fine.

<!--more-->

## Issue

Sometimes we need to use [Google Trends](https://trends.google.com/trends/) to find the search trends for key words and embed the result into our own html page,we can click `<>` icon on the right top of each chart to get the embed source code that can be integrated into html page:

For example, we can use it to find the search trends for key words: `Podman`,`Docker` and `Kubernetes` in the past 5 years worldwide,below are the screenshots for them:

* `Podman` search trend

  result:

  ![Podman search trends](/blog_img/web/embed-multiple-google-trends-charts-into-single-html-page/podman-search-trend.png "Podman search trends")

  embed code:

  ```html
  <script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/3728_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Podman","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Podman","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>
  ```

* `Docker` search trend

  result:

  ![Docker search trends](/blog_img/web/embed-multiple-google-trends-charts-into-single-html-page/docker-search-trend.png "Docker search trends")

  embed code:

  ```html
  <script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/3728_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Docker","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Docker","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>
  ```

* `Kubernetes` search trend

  result:

  ![Kubernetes search trends](/blog_img/web/embed-multiple-google-trends-charts-into-single-html-page/docker-search-trend.png "Kubernetes search trends")

  embed code:

  ```html
  <script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/3728_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Kubernetes","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Kubernetes","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>
  ```

If we put the embed codes into single page, usually we can put these three code blocks into html page directly:

 ```html
<!DOCTYPE html>
<html>
<head>
   <title>Google Trends Charts Embed Test</title>
</head>
<body>

<!-- Podman -->
<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/3728_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Podman","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Podman","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

<!-- Docker -->
<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/3728_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Docker","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Docker","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

<!-- Kubernetes -->
<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/3728_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Kubernetes","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Kubernetes","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

</body>
</html>
 ```

However,if we test it inside a web server we will find **only the first one can works fine,the others are not.**

![Only the first chart is working](/blog_img/web/embed-multiple-google-trends-charts-into-single-html-page/only-the-first-one-working.png "Only the first chart is working")

Let's modify the code and make only one chart left,we will find that **it will works fine where there are only one chart!**

We can check it at [**live demo with three charts not working**](https://jsfiddle.net/kt82567q/) and [**live demo with one chart working**](https://jsfiddle.net/kt82567q/1/).

## Solution

When the page has three charts,we can check the browser console and will find that there are no error inside it, there must be something wrong with my code!

After search on the Internet I found a solution[^1],the reason is that **we have imported duplicated`embed_loader.js`,it only needs imported one!**

Change code as below

```html
<!DOCTYPE html>
<html>
<head>
   <title>Google Trends Charts Embed Test</title>
</head>
<body>

<!-- Podman -->
<!-- embed_loader.js only needs to be import once -->    
<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/3728_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Podman","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Podman","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

<!-- Docker -->
</script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Docker","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Docker","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

<!-- Kubernetes -->
</script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Kubernetes","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"date=today%205-y&q=Kubernetes","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

</body>
</html>
```

Test it again we will find all the charts working fine,that's all!

![All charts working fine](/blog_img/web/embed-multiple-google-trends-charts-into-single-html-page/all-charts-working-file.png "All charts working fine")



We can check it at [**live demo with three charts working**](https://jsfiddle.net/upgnjfkx/)

[^1]: [Google Trends embed graphs not working in WordPress 6.2.2](https://wordpress.org/support/topic/google-trends-embed-graphs-javascript-not-working-in-wordpress-6-2-2/)