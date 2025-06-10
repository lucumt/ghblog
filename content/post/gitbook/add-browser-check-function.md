---
title: "在GitBook中添加浏览器类型与版本检测"
date: 2025-06-09T18:32:17+08:00
lastmod: 2025-06-09T18:32:17+08:00
draft: false
keywords: ["GitBook","浏览器","Chrome"]
description: "对GitBook中开发插件，实现检测浏览器类型与版本，避免使用低版本时不支持部分高级特性的使用"
tags: ["GitBook"]
categories: ["工具使用"]
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

在部门使用`GitBook`的过程中，发现有些同事使用旧版的`Chrome`浏览器会造成`GitBook`页面显示不正常，故开发了一款插件对当前使用的浏览器类型和版本进行检测，强制使用不能低于特定版本的`Chrome`浏览器，否则会弹出提示信息。

<!--more-->

# 问题背景

当使用`Chrome`浏览器版本较旧时(如版本为`88.0.4324.104`），打开`GitBook`后页面中的部分样式会显示不正常，类似下图所示，如左侧菜单以及提示插件等都不能正常渲染，影响使用体验。

![GitBook在不同浏览器中的显示效果对比](/blog_img/gitbook/add-browser-check-function/gitbook-render-in-different-browsers.png "GitBook在不同浏览器中的显示效果对比")

出现此种问题的原因是`GitBook`中使用了一些高级的`CSS`和`HTML`特定，而低版本的`Chrome`浏览器不支持这些特定，从而导致此问题。

对于非专业的`GitBook`开发维护人员而言，出现此问题后通常不知道问题原因以及如何处理，有必要给出醒目的提示，告知必须升级浏览器。

而`GitBook`本身提供了灵活强大的插件开发功能，基于此，可通过开发对应的插件来实现强制检测与提醒。

# 实现思路

要实现检测浏览器版本，关键是需要在页面每次加载时触发相关事件进行浏览器版本检测，类似`jQuery`中的[ready()](https://api.jquery.com/ready/)函数，在`GitBook`中对应的事件回调函数为`page.change`：

```javascript
gitbook.events.bind('page.change', function() {
    // 此处编写相关的代码
});
```

找到入口之后，接下来的事情就是检测浏览器类型与版本，这种事情可直接咨询DeepSeek，我们只需整合相关的代码即可实现目的。

`GitBook`支持在`book.js`中的`pluginsConfig`下添加相关配置实现自定义展现，基于此功能可实现让要检测的`Chrome`版本以及相关的提示信息能动态配置，其对应的事件回调函数为`start`。

```javascript
gitbook.events.bind('start', function(e, config) {
   // book.js中的pluginsConfig下的所有信息都在config中
   // 可将其赋值给全局变量，然后在其它事件回调函数中使用
   console.log(config);
});
```

# 相关源码

完成代码参见[**gitbook-plugin-browser-check-fox**](https://github.com/gitbook-plugin-fox/gitbook-plugin-browser-check-fox)，核心代码如下：

`book.js`的配置代码：

```javascript
pluginsConfig: {
    'browser-check-fox': {
        'enabled': true,
        'chrome-required-version': 126,
        'chrome-required-message': '只支持使用Chrome浏览器',
        'chrome-required-version-message': '当前浏览器版本为${currentVersion},需要升级到${requiredVersion}'
    }
}
```

`index.js`代码：

```javascript
module.exports = {
    book: {
		assets:'./',
        js: ["check.js"]
    }
};
```

`check.js`代码：

```javascript
require(['gitbook', 'jQuery'], function(gitbook, $) {

    var browserCheckConfig = {};
    gitbook.events.bind('start', function(e, config) {
        let defaultConfig = {
            enabled: true,
            'chrome-required-message': 'Only Chrome is supported to access this website',
            'chrome-required-version': '126',
            'chrome-required-version-message': 'The version of your chrome brower is ${currentVersion},upgrade to ${requriedVersion},please'
        };
        let customConfig = config['browser-check-fox'];
        for (let key of Object.keys(defaultConfig)) {
            browserCheckConfig[key] = customConfig[key] ?? defaultConfig[key];
        }
    });
    gitbook.events.bind('page.change', function() {
        checkBrowserVersion(browserCheckConfig);
    });

});

function checkBrowserVersion(config) {
    if (!config['enabled']) {
        return;
    }
    const userAgent = navigator.userAgent;
    let isChrome = false;
    let currentVersion = 0;
	let requiredVersion = config['chrome-required-version'];

    // 使用正则表达式检测 Chrome 浏览器（排除 Edge、Opera 等基于 Chromium 的浏览器）
    const chromeRegex = /(Chrome)\/(\d+)/;
    const edgeRegex = /(Edg|Edge)\//;
    const operaRegex = /(OPR)\//;

    // 排除 Edge 和 Opera
    if (!edgeRegex.test(userAgent) && !operaRegex.test(userAgent)) {
        const chromeMatch = userAgent.match(chromeRegex);
        if (chromeMatch) {
            isChrome = true;
            currentVersion = parseInt(chromeMatch[2]); // 提取主版本号
        }
    }

    // 判断并提示
    if (!isChrome) {
        alert(config['chrome-required-message']);
    } else if (currentVersion < requiredVersion) {
        let message = config['chrome-required-version-message'];
        message = message.replace('${currentVersion}', currentVersion);
        message = message.replace('${requiredVersion}', requiredVersion);
        alert(message);
    }
}
```

# 使用效果

插件包地址参见[**gitbook-plugin-browser-check-fox**](https://www.npmjs.com/package/gitbook-plugin-browser-check-fox)，其使用方式同其它的`GitBook`插件类似。

使用非`Chrome`浏览器时的提示效果如下

![使用非Chrome浏览器时提示信息](/blog_img/gitbook/add-browser-check-function/browser-type-not-match-alert.png "使用非Chrome浏览器时提示信息")

使用`Chrome`浏览器版本较低时的提示效果如下

![Chrome浏览器版本太旧提示](/blog_img/gitbook/add-browser-check-function/browser-version-too-old-alert.png "Chrome浏览器版本太旧提示")