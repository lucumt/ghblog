+++

author = "飞狐"
categories = ["其它"]
toc = false
tags = ["Mediawiki"]
date = "2016-03-18T23:14:59+08:00"
description = "简要记录如何在Mediawiki中添加回到顶部的方法，以方便阅读篇幅太长的内容"
keywords = ["Mediawiki"]
title = "Mediawiki添加回到顶部的方法"

+++

[**Mediawiki**](https://www.mediawiki.org/wiki/MediaWiki)是[**维基百科**](https://www.wikipedia.org/)系统所采用的框架，适合于需要快速搭建知识分享的场合。采用[**Mediawiki**](https://www.mediawiki.org/wiki/MediaWiki)生成的知识共享平台和[**维基百科**](https://www.wikipedia.org/)的操作与使用类似，都支持采用[**Markdown**](https://zh.wikipedia.org/zh-cn/Markdown)语法来编辑。在有些时候，某些词条的内容很长，使得浏览器出现了滚动条，如果能仿照微博等网站添加一个**回到顶部**的功能，将会给我们的使用带来很大的便利，本文介绍一种实现方法：

<!--more-->

* 以 [**Mediawiki**](https://www.mediawiki.org/wiki/MediaWiki) 管理员身份登录mediawiki,在搜索栏输入`MediaWiki:Common.js`,然后输入如下代码并保存：

    ``` javascript
    /* 此处的JavaScript将加载于所有用户每一个页面。 */
    $(window).scroll(function(){
       if($(window).scrollTop()>100){
        $(".back-to-top").fadeIn(1000);
      }else{
        $(".back-to-top").fadeOut(1000);
      }
    });
    ```
* 在`mediawiki\skins\Vector.php`中的第252行添加如下代码： 

     ```html
       <div class="back-to-top" onClick="$('html,body').animate({scrollTop:0},500);">
          <span>返回顶部</span>
       </div>
     ```

* 在`mediawiki\skins\vector\screen.css`的最后添加如下代码：

    ```css
    .back-to-top {
        position: fixed;
        bottom: 6em;
        right: 3em;
        background-color: rgba(46, 46, 46, 0.8);
        text-align: center;
        padding: 5px 6px;
        color: #eee;
        -webkit-border-radius: 3px;
        -moz-border-radius: 3px;
        border-radius: 3px;
        cursor: pointer;
        display: none;
    }

    .back-to-top:hover {
        background: rgba(0, 221, 255, 0.8);
    }
    ```

* 当页面的高度超出限制时，就会出现“返回顶部”的悬浮框，效果图如下：  

    ![Back to top.PNG](/blog_img/web/mediawiki-back-to-top/back-to-top.png "返回顶部示例图片")