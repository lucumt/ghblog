---
title: "给GitBook生成的html页面中添加水印支持"
date: 2025-01-16T13:38:21+08:00
lastmod: 2025-01-16T13:38:21+08:00
draft: false
keywords: ["GitBook","水印"]
description: "基于已有的js水印插件给GitBook开发一款水印插件，可在所有页面显示水印且能动态的开启或关闭"
tags: ["GitBook"]
categories: ["工具使用"]
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
borderImage: false
codeTabSeperator: "::"
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

简要介绍个人基于[**watermark-js-plus**](https://zhensherlock.github.io/watermark-js-plus/)开发的一款`GitBook`[**水印插件**](https://github.com/gitbook-plugin-fox/gitbook-plugin-watermark-fox)，可用于给`GitBook`所有页面添加自定义水印，且能根据需求动态的开启或关闭。
<!--more-->

## 原始插件

原始插件[watermark-js-plus](https://zhensherlock.github.io/watermark-js-plus/)的使用很简单，参考[使用入门](https://zhensherlock.github.io/watermark-js-plus/guide/getting-started.html#cdn)只需要基于下述步骤就能快速的实现一个水印效果

1. 引入类似如下js

   ```html
   <script src="https://cdn.jsdelivr.net/npm/watermark-js-plus/dist/index.iife.min.js"></script>
   ```

2. 添加下述代码

    ```javascript
   const watermark = new WatermarkPlus.Watermark({
     content: 'hello my watermark',
     width: 200,
     height: 200
   })
   watermark.create();
    ```

3. 展示效果如下，可参见[完整demo](https://jsfiddle.net/49c02any/)

   ![水印插件显示效果](/blog_img/gitbook/add-watermark-in-gitbook/watermark-plugin-demo.png "水印插件显示效果")

## GitBook插件开发

前述的插件是基于纯`JavaScript`实现的，不能直接在`GitBook`中使用，需要遵循`GitBook`的插件规范开发相应的插件。

由于原始插件是在页面加载完毕后执行的，故在`GitBook`中同样需要类似的实现，需要涉及到`GitBook`的事件函数。由于网络上关于`GitBook`的插件开发资料较少，可参考[这篇文章](https://gitbook.whyun.com/plugins/)或[这篇文章](https://www.jianshu.com/p/22f5f1a107a3)，也可参考其它的`GitBook`插件是如何开发的，从中学习相关事件和钩子函数(Hooks)的使用。

为了让插件的使用更灵活，如可动态开启或关闭水印，动态设置水印内容等，可通过`pluginsConfig`来动态配置插件信息。



相关开发步骤如下：

1. 在`package.js`中添加类似如下类容，其中的信息可根据个人实际情况填写，该文件主要用于`npm`发布文件时使用

   ```javascript
   {
     "name": "gitbook-plugin-watermark-fox",
     "description": "Add watermark to GitBook document to protect copyright",
     "main": "index.js",
     "version": "0.0.3",
     "author": {
       "name": "lucumt",
       "email": "lucumt@gmail.com"
     },
     "keywords": [
       "gitbook",
       "gitbook-plugin",
   	"custom",
   	"watermark"
     ],
     "engines": {
       "gitbook": ">=2.4.3"
     },
     "homepage": "https://github.com/gitbook-plugin-fox/gitbook-plugin-watermark-fox",
     "repository": {
       "type": "git",
       "url": "https://github.com/gitbook-plugin-fox/gitbook-plugin-watermark-fox.git"
     },
     "repository": {
       "type": "git",
       "url": ""
     },
     "license": "Apache 2",
     "dependencies": {
     }
   }
   ```

2. 在插件根目录下创建一个名为`index.js`的文件，内容如下

   ```javascript
   module.exports = {
      book: {
         assets: './assets',
         js: [
            'watermark.min.js',
            'plugin.js'
         ],
      }
   };
   ```

3. 在插件根目录下创建一个名为`assets`的文件夹，然后将原始插件的js文件放入该目录下，文件名称可随意命名

4. 在`assets`目录下创建一个名为`plugin.js`的文件，并写入如下内容。该文件主要包含两部分，从第5行到第17行用于读取`pluginsConfig`下`watermark-fox`里面的相关配置，识别水印是否开启以及对应的配置项，第18行到第23行则用于根据配置项创建水印并渲染。

   ```javascript {data-line="5,18"}
   require([
      'gitbook'
   ], function (gitbook) {
      var watermarkConfig = {};
      gitbook.events.bind('start', function (e, config) {
         let defaultConfig = {
            content: 'www.gitbook.com',
            width: 200,
            height: 200,
            fontColor: '#d0d0d0',
            enable: true
         };
         let customConfig = config['watermark-fox'];
         for (let key of Object.keys(defaultConfig)) {
            watermarkConfig[key] = customConfig[key] ?? defaultConfig[key];
         }
      });
      gitbook.events.bind('page.change', function (e) {
         if (watermarkConfig.enable) {
            const watermark = new WatermarkPlus.Watermark(watermarkConfig);
            watermark.create();
         }
      });
   });
   ```

5. 至此，插件编写完毕，可通过`npm link`在本地直接测试，也可通过`npm publish`发布到公共仓库测试。

## 使用展示

可基于下述步骤验证前面开发的插件：

1. 在`book.js`中添加类似如下代码，出于篇幅考虑，`book.js`中多余的代码已省略

   ```javascript
   module.exports = {
       plugins: [
           'watermark-fox' 			  // 加载该插件
       ],
       pluginsConfig: {
           "watermark-fox": {
               "content": "www.foo.com", // 水印内容
               "width": 200,             // 单个水印的宽度
               "height": 200,            // 单个水印的高度
               "fontColor": "#d0d0d0",   // 水印字体颜色
               "enable": true            // 是否开启该插件
           },
       }
   };
   ```

2. 执行`gitbook serve`启动`GitBook`程序

3. 在浏览器中打开`http://127.0.0.1:4000`，可看见所有页面均显示水印，插件正常工作。

   ![GitBook水印显示效果](/blog_img/gitbook/add-watermark-in-gitbook/gitbook-watermark-demo.png "GitBook水印显示效果")