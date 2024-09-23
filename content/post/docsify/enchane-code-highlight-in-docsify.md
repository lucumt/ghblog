---
title: "对Docsify中的代码高亮显示进行增强"
date: 2023-09-22T10:13:56+08:00
lastmod: 2023-09-22T10:13:56+08:00
draft: false
keywords: ["docsify","行号","代码高亮"]
description: "简要记录如何对Docsify中的代码高亮显示进行增强，主要包括行间距样式改进以及显示行号的增强"
tags: ["docsify"]
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

记录下如何给[**docsify**](https://docsify.js.org/)添加多语言的代码高亮支持，以及在代码高亮的同时能显示行号，便利使用。

<!--more-->

## 环境准备

需要预先安装`nodejs`，之后在终端执行下述命令

```bash
npm i docsify-cli -g
docsify init ./docs
docsify serve ./docs
```

之后可通过`http://localhost:3000`来访问默认的模板，给默认生成的`README.md`文件分别添加`javascript`、`java`、`golang`代码块后显示结果类似如下

![docsify默认的代码高亮显示效果](/blog_img/docsify/enchane-code-highlight-in-docsify/docsify-default-code-highlight-support.png  "docsify默认的代码高亮显示效果")

从上图中可发现如下几个问题：

1. `javascript`代码有高亮显示，而`java`和`golang`代码没有高亮显示
2. `javascript`代码虽然支持高亮显示，但是没有显示行号
3. 代码块与真正的代码内容之间的间距过大，占用过多空间也不便于阅读

## 代码高亮

在[**Language highlighting**](https://docsify.js.org/#/language-highlight)中有如下说明

![docsify代码高亮说明](/blog_img/docsify/enchane-code-highlight-in-docsify/docsify-highlight-support.png  "docsify代码高亮说明")

可知其默认支持的代码高亮语言有限，其它的需要手工添加，在[**prismjs CDN files**](https://cdn.jsdelivr.net/npm/prismjs@1/components/)可查看全部支持的语言列表，在其生成的`index.html`文件中添加相关的高亮文件

```html
<script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-go.min.js"></script>
<script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-java.min.js"></script>
```

保存配置后，可发现浏览器中的页面已经修改，`java`和`golang`已经添加了高亮支持

![docsify添加代码高亮](/blog_img/docsify/enchane-code-highlight-in-docsify/docsify-add-code-highlight-support.png  "docsify添加代码高亮")

前述第1个问题解决，余下的2个仍未解决。

## 样式改进

在`index.html`中添加如下`css`代码

```css
.markdown-section pre > code {
    padding: 5px;
}
```

此时可发现代码块的间距已经显著减少，问题2解决。

![docsify代码高亮样式改进](/blog_img/docsify/enchane-code-highlight-in-docsify/docsify-code-highlight-style-enchance.png  "docsify代码高亮样式改进")

## 显示行号

显示行号是代码高亮的一个刚需功能，不太明白为啥`docsify`的开发者一直没有整合这个功能。

简单搜素下就发现有人已经提过此类问题 **[Prismjs supports line numbers and sepcific line highlight](https://github.com/docsifyjs/docsify/issues/771#issuecomment-655982606)** 根据该链接的回复对`index.js`做一些修改。

将默认的渲染行为修改如下：

```javascript
window.$docsify = {
    name: '',
    repo: '',
    markdown: {
        renderer:{
            code: function(code, lang) {
                if(lang =='html'){
                    code = code.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
                }
                return ('<pre data-lang="'+lang+'"><code class="line-numbers language-'+lang+'">'+code+'</code></pre>')
            }
        }	  
    },
    plugins: [
        function (hook, vm) {
            hook.doneEach(function (html) {                
                Prism.highlightAll();
            })
        }
    ]

}
```

添加如下`css`样式

```css
.markdown-section pre[data-lang] {
    overflow: auto !important;
}

.markdown-section pre[data-lang] code {
    overflow: visible;
}

.line-numbers .line-numbers-rows {
    border-right : 0px solid white;
    /* Fix paddings to align with code.*/
    padding: 5px; /* Same as code block */
}
```

分别引入相关的`css`和`js`文件

```html
<link rel="stylesheet" href="//cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-numbers/prism-line-numbers.min.css">
<script src="//cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-numbers/prism-line-numbers.min.js"></script>
```

之后查看`docsify`页面，效果如下，至此问题3也解决，顺利达成目的！

![docsify代码高亮且有行号](/blog_img/docsify/enchane-code-highlight-in-docsify/docsify-code-highlight-with-line-numbers.png  "docsify代码高亮且有行号")

## 后记

### 框架切换

由于`docsify`是将所有的`markdown`文件全部加载并渲染成单个`html`文件，导致其在初次打开时很卡顿，渲染完成后就很流畅。但每次打开浏览器访问`docsify`时都需要花时间进行初次渲染，尤其是随着`markdown`文件越来越多，渲染耗时也越来越多，严重影响体验。

后续处于便于使用以及便于扩展、维护的角度考虑，最终将`docsify`替换为了`GitBook`，详情参见[**GitBook插件中实现从多个不同的文件夹下加载css和js文件**](/post/gitbook/using-docker-to-build-gitbook-with-gitlab-runner/)。

### 参考代码

完整的`index.html`代码如下

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="description" content="Description">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-numbers/prism-line-numbers.min.css">
  <style type="text/css">	
	.markdown-section pre[data-lang] {
		overflow: auto !important;
	}
	
	.markdown-section pre[data-lang] code {
		overflow: visible;
	}

	.line-numbers .line-numbers-rows {
		border-right : 0px solid white;
		/* Fix paddings to align with code.*/
		padding: 5px; /* Same as code block */
	}
	
	.markdown-section pre > code {
		padding: 5px;
	}
  </style>
</head>
<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      name: '',
      repo: '',
	  markdown: {
            renderer:{
				code: function(code, lang) {
				    if(lang =='html'){
					  code = code.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
					}
					return ('<pre data-lang="'+lang+'"><code class="line-numbers language-'+lang+'">'+code+'</code></pre>')
				}
			}	  
	 },
	 plugins: [
		  function (hook, vm) {
			  hook.doneEach(function (html) {                
				Prism.highlightAll();
			  })
		  }
	  ]

    }
  </script>
  <!-- Docsify v4 -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-go.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-java.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-numbers/prism-line-numbers.min.js"></script>
</body>
</html>
```

