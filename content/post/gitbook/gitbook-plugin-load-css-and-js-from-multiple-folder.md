---
title: "GitBook插件中实现从多个不同的文件夹下加载css和js文件"
date: 2023-12-10T16:21:12+08:00
lastmod: 2023-12-10T16:21:12+08:00
draft: false
keywords: ["gitbook","plugin","插件"]
description: "GitBook插件中实现从多个不同的文件夹下加载css和js文件,为后续的自定义插件实现提供参考"
tags: ["gitbook"]
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

在[**在Docker中构建GitBook并整合GitLab Runner的使用经验分享**](/post/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/)一文中，自己在`Docker`环境下基于`GitLab Runner`实现了`GitBook`的自动构建，同时基于实际使用需求添加了很多`GitBook`插件，在此过程中积累了一些`GitBook`插件的开发与使用经验。

本文简要介绍自己在`GitBook`插件定制化修改过程中如何实现从多个不同的文件夹下加载`js`和`css`文件的实现过程。

<!--more-->

## 问题背景

由于`GitBook`官方默认的代码高亮插件[**highlight**](https://github.com/GitbookIO/plugin-highlight)使用的是[**highlightjs**](https://highlightjs.org/)，其功能没有[**prismjs**](https://prismjs.com/)完善，故最初将`GitBook`的代码高亮采用`prismjs`实现，对应的插件为[**gitbook-plugin-prism**](https://github.com/gaearon/gitbook-plugin-prism)，其使用方式也很简单，只需要在`book.json`中添加如下配置:

```json
{
  "plugins": ["prism", "-highlight"]
}
```

该插件使用起来一切正常，由于部门`GitBook`的一大用途是记录设计方案与实现的代码，经常会有多个功能相似的对比代码进行展示，占用了大量的空间篇幅且阅读也不是很方便，需要添加一个[**tab**](https://www.w3schools.com/howto/howto_js_tabs.asp)效果对其进行分组归类。

在网上搜索相关的`GitBook`插件后，主要有[**gitbook-plugin-codegroup**](https://github.com/lwhiteley/gitbook-plugin-codegroup)以及[**gitbook-plugin-codetabs**](https://github.com/GitbookIO/plugin-codetabs)，它们的对比如下：

|                |             `gitbook-plugin-codegroup`             |      `gitbook-plugin-codetabs`      |
| :------------: | :------------------------------------------------: | :---------------------------------: |
| `代码高亮实现` |                    `highlight`                     |               `prism`               |
| `代码写入方式` | 进行`markdown`中代码块原生语法，`typora`展示更方便 | `GitBook`代码块，`typora`展示不友好 |
| `插件改造难度` |                         高                         |                 低                  |

综合对比后，决定采用`gitbook-plugin-codetabs`结合`gitbook-plugin-prism`两者结合，同时代码块的输入还是采用类似 **```** 包裹的方式以便与`markdown`和`typora`的使用方式保持一致。

确定好方案后，接下来就需要将这两个插件组合到一起，查看它们的源码，发现都各自引入了静态文件

![gitbook插件中都分别引入了静态文件](/blog_img/gitbook/gitbook-plugin-load-css-and-js-from-multiple-folder/gitbook-plugins-assets-config.png "gitbook插件中都分别引入了静态文件")

自己首先想到的是添加多个`assets`然后在`js`和`css`使用完整路径，类似如下：

```js
module.exports = {
    book: {
        assets: ['./assets1','./assets2'],
        css: [
            'codetabs1.css','codetabs2.css'
        ],
        js: [
            'codetabs1.js','codetabs2.js'
        ]
    },
```

但此种方式明显不可行，通过`gitbook serve`启动之后，终端控制台提示类似如下错误

```bash
TypeError [ERR_INVALID_ARG_TYPE]: The "path" argument must be of type string. Received an instance of List
```

很明显`assets`的值只能为单个字符串，不能为数组。

接下来尝试将`assets` 配置去掉，改为类似如下代码

```javascript
book: {
   css: [
       './assets1/codetabs1.css','./assets2/codetabs2.css'
   ],
   js: [
       './assets1/codetabs1.js','./assets2/codetabs2.js'
   ]
},
```

此时通过`gitbook serve`启动时，终端控制台提示一切正常，但访问对应的页面时，显示效果不正常，通过`Chrome`调试工具可发现上述的`js`和`css`都显示为`404`，都未能正常加载，导致页面显示不正常。

看来`assets`属性不能省略，且不能以数组方式配置，此时很容易想到的是将这些静态资源文件都放置到一个文件夹下，不就解决了此问题？

理论上确实可行，但是理想很丰满，现实很骨感，此种方式并不能满足自己的实际需求！

原因为`gitbook-plugin-prism`插件的`index.js`中有如下代码，通过此代码可知`gitbook-plugin-prism`中是通过去`npm`仓库中获取`prismjs`的源文件，而不是直接将`primsjs`下载到该插件的资源文件目录下，通过这种方式可实现`gitbook-plugin-prism`与`prismjs`的解耦，且能快速的升级`prismjs`，是一个合理且优秀的设计！

作为一个对代码编写要求很高的人，当前要将别人优秀的习惯保持下来，这样的话就不能将代码分组的静态文件与`prismjs`代码高亮相关的文件混合在一起，问题产生！

```javascript
function getAssets() {

  var cssFiles = getConfig(this, 'pluginsConfig.prism.css', []);
  var cssFolder = null;
  var cssNames = [];
  var cssName = null;

  if (cssFiles.length === 0) {
    cssFiles.push('prismjs/themes/prism.css');
  }

  cssFiles.forEach(function(cssFile) {
    var cssPath = require.resolve(cssFile);
    cssFolder = path.dirname(cssPath);
    cssName = path.basename(cssPath);
    cssNames.push(cssName);
  });

  return {
    assets: cssFolder,
    css: cssNames
  };
}
```

## 解决方案

既不能同时支持多个`assets`属性，自己又不想破坏`prismjs`本身的结构，怎么办？

一开始自己通过`Google`和`GitHub`去查找相关的资料，没有找到自己想要的东西，后来用`ChatGPT`连续切换了好几次提示词，结果答案都是错的.

> PS:个人鄙视在编程中遇到问题无脑使用`ChatGPT`,个人观点是要明白其底层原理，找到合适的使用方法。`ChatGPT`是基于训练的，网络上相关资料越丰富，`ChatGPT`训练的材料也就越多，其给出的答案也就越准确，反之，对于新出现的技术，网上资料很少或者`ChatGPT`来不及训练，此时就不能给出准确的答案了。

通过前述方式没有找到自己想要的答案，没办法只能在`Stackoverflow`上用英文提问，希望能有人给回复，遗憾的提出问题之后一周也没人回复，只能另想它法了。

分析`GitBook`中插件的生成结果，发现要想某个静态资源文件能够被访问到，其必须在`_book`目录下对应的插件中存在，之前的去掉`assets`后`gitbook serve`不报错，但是`js`和`css` 加载出现`404`即是这个原因。

![gitbook中可访问文件路径](/blog_img/gitbook/gitbook-plugin-load-css-and-js-from-multiple-folder/gitbook-plugin-resource-location.png "gitbook中可访问文件路径")

找到问题原因后，接下来自己尝试手工把相关的`js`和`css`文件加到`_book`下对应的插件目录中，此时可通过浏览器地址直接访问，不会出现`404`问题，但是相关的静态资源文件仍然没有加载，在生成的html源码中也没有看见。

![gitbook资源文件没有加载](/blog_img/gitbook/gitbook-plugin-load-css-and-js-from-multiple-folder/gitbook-plugin-resource-files-not-loaded.png "gitbook资源文件没有加载")

问题原因为我们没有在下述代码中引入对应的`js`和`css`文件。

```javascript
book: {
    assets: './assets',
        css: [
            'codetabs.css'
        ],
        js: [
            'codetabs.js'
        ]
},
```

要想文件被加载，则必须要引入相关文件，要想文件能被正常访问，在`_book`下对应的插件目录中必须要有对应的文件。

至此，问题的解决方案找到！

引入文件很容易实现，直接在数组中添加对应的文件即可，但如何引入相关文件呢？经过多处查看代码后，自己在[**index.js**](https://github.com/gaearon/gitbook-plugin-prism/blob/master/index.js#L151)中找到了答案

```javascript
try {
    fs.writeFileSync(outputFile, fs.readFileSync(inputFile));
} catch (e) {
    console.warn('Failed to write prism-ebook.css. See https://git.io/v1LHY for side effects.');
    console.warn(e);
}
```

利用相关的`JavaScript` API手工将文件写入`_book`下对应插件的目录中！之后将`assets`的值设置为对应插件的名称，既能同时访问多个文件！



经过多次测试，最终捣鼓出可用的代码，在不破坏`prismjs`完整性的同时还能加载插件自身的`js`和`css`文件，完整代码参见[**gitbook-plugin-prism-codetab-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-prism-codetab-fox/)

核心的`index.js`代码如下，主要是通过`syncFile`函数进行文件写入：

```javascript
var Prism = require('prismjs');
var languages = require('prismjs').languages;
var path = require('path');
var fs = require('fs');
var cheerio = require('cheerio');
var mkdirp = require('mkdirp');
var codeBlocks = require('gfm-code-blocks');
var trim = require('lodash/trim');
const includes = require('lodash/includes');
const get = require('lodash/get');

var DEFAULT_LANGUAGE = 'markup';
var DEFAULT_CODE_TAB_SEPERATOR = '::';
var MAP_LANGUAGES = {
    'py': 'python',
    'js': 'javascript',
    'rb': 'ruby',
    'cs': 'csharp',
    'sh': 'bash',
    'html': 'markup'
};

// Base languages syntaxes (as of prism@1.6.0), extended by other syntaxes.
// They need to be required before the others.
var PRELUDE = [
    'markup-templating', 'clike', 'javascript', 'markup', 'c', 'ruby', 'css',
    // The following depends on previous ones
    'java', 'php'
];
PRELUDE.map(requireSyntax);

/**
 * Load the syntax definition for a language id
 */
function requireSyntax(lang) {
    require('prismjs/components/prism-' + lang + '.js');
}

function getConfig(context, property, defaultValue) {
    var config = context.config ? /* 3.x */ context.config : /* 2.x */ context.book.config;
    return config.get(property, defaultValue);
}

function isEbook(book) {
    // 2.x
    if (book.options && book.options.generator) {
        return book.options.generator === 'ebook';
    }

    // 3.x
    return book.output.name === 'ebook';
}

function getAssets() {

    var cssFiles = getConfig(this, 'pluginsConfig.prism.css', []);
    var cssFolder = null;
    var cssNames = [];
    var cssName = null;

    if (cssFiles.length === 0) {
        cssFiles.push('prismjs/themes/prism.css');
    }

    cssFiles.forEach(function(cssFile) {
        var cssPath = require.resolve(cssFile);
        cssFolder = path.dirname(cssPath);
        cssName = path.basename(cssPath);
        cssNames.push(cssName);
    });

    cssNames.push('codetab/codetab.css');
    return {
        assets: cssFolder,
        css: cssNames,
        js: ['codetab/codetab.js']
    };
}

function syncFile(book, outputDirectory, outputFile, inputFile) {
    outputDirectory = path.join(book.output.root(), '/gitbook/gitbook-plugin-prism-codetab-fox/' + outputDirectory);
    outputFile = path.resolve(outputDirectory, outputFile);
    inputFile = path.resolve(__dirname, inputFile);
    mkdirp.sync(outputDirectory);

    try {
        fs.writeFileSync(outputFile, fs.readFileSync(inputFile));
    } catch (e) {
        console.warn('Failed to write ' + inputFile);
        console.warn(e);
    }

}

module.exports = {
    book: getAssets,
    ebook: function() {

        // Adding prism-ebook.css to the CSS collection forces Gitbook
        // reference to it in the html markup that is converted into a PDF.
        var assets = getAssets.call(this);
        assets.css.push('prism-ebook.css');
        return assets;
    },
    blocks: {
        code: // xxxx
        codetab: // xxxx
    },
    hooks: {

        // Manually copy prism-ebook.css into the temporary directory that Gitbook uses for inlining
        // styles from this plugin. The getAssets() (above) function can't be leveraged because
        // ebook-prism.css lives outside the folder referenced by this plugin's config.
        //
        // @Inspiration https://github.com/GitbookIO/plugin-styles-less/blob/master/index.js#L8
        init: function() {

            var book = this;

            syncFile(book, 'codetab', 'codetab.js', './codetab/codetab.js');
            syncFile(book, 'codetab', 'codetab.css', './codetab/codetab.css');

            // If failed to write prism-ebook.css. See https://git.io/v1LHY for side effects.
            if (isEbook(book)) {
                syncFile(book, '', 'prism-ebook.css', './prism-ebook.css');
            }

        },
        page: // xxx
    }
};
```

自己改造后的插件名为[**gitbook-plugin-prism-codetab-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-prism-codetab-fox)，将该插件在`book.json`中启用并运行`gitbook install && gitbook serve`命令后，可以看见在对应的`GitBook`工程下分别将`prismjs`和`codetab`相关的插件都安装到`node modules`模块下。

![gitbook node modules列表](/blog_img/gitbook/gitbook-plugin-load-css-and-js-from-multiple-folder/gitbook-node-modules.png "gitbook node modules列表")

分别检查这两个`js`模块，可发现其内容都正常

![gitbook node modules文件列表](/blog_img/gitbook/gitbook-plugin-load-css-and-js-from-multiple-folder/gitbook-node-modules-files.png "gitbook node modules文件列表")

检查`_book`目录，发现生成的插件同时包含`prismjs`以及` codetab`相关的文件，此时该插件已经将相关的内容合并了。

![gitbook生成的插件](/blog_img/gitbook/gitbook-plugin-load-css-and-js-from-multiple-folder/gitbook-generated-plugin-files.png "gitbook生成的插件")

 查看`GitBook`中生成的`HTML`页面源码，可发现相关的静态文件均正常加载，至此问题解决！

![gitbook加载多个文件](/blog_img/gitbook/gitbook-plugin-load-css-and-js-from-multiple-folder/gitbook-plugin-load-multiple-files.png "gitbook加载多个文件")