---
title: "在Hugo中利用Prism进行代码高亮"
date: 2024-07-30T18:31:44+08:00
lastmod: 2024-07-30T18:31:44+08:00
draft: false
keywords: ["hugo","prism","chroma","highlight","代码高亮"]
description: "简要介绍如何使用Prism来替换Hugo博客默认的代码高亮实现Chroma，实现支持更多的语言和更好的识别"
tags: ["hugo"]
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

简要介绍如何在基于[**Even**](https://github.com/olOwOlo/hugo-theme-even)主题的[**Hugo**](https://gohugo.io)博客中采用[**Prism**](https://prismjs.com)替换默认的[**Chroma**](https://github.com/alecthomas/chroma)进行代码高亮显示，实现支持更多的语言和更好的识别。

<!--more-->

## 背景

个人博客采用`Hugo`编写已经有年头了，由于`Hugo`是基于`Golang`编写的，而`Chroma`也是基于`Golang`编写的，故`Hugo`采用了`Chroma`作为其默认的代码高亮显示实现[^1]。自己的个人博客搭建完毕之后一直采用该默认实现。

之后的几年虽然自己的博客主题切换成了`Even`主题，也伴随着各种优化，但对于代码高亮这块却一直没变化，一方面是自己觉得其显示效果还行，没找到更适合自己口味的显示样式，就和代码开发一样，能正常运行的代码就不要随便优化重构(虽然自己很喜欢代码重构)，另一方面则是由于代码高亮切换，改动时涉及的底层页面较多。

近期由于部门内部要搭建基于`GitBook`的知识库，在此过程中采用的语法高亮为`Prism`，经过一段时间的时候之后个人感觉`Prism`对于代码高亮显示的支持更强，对它们的关键指标进行对比后，初步决定采用`Prism`替换`Chroma`。

|                |              Prime               |                Chroma                |
| -------------- | :------------------------------: | :----------------------------------: |
| **GitHub地址** | https://github.com/PrismJS/prism | https://github.com/alecthomas/chroma |
| **Star数**     |              12.2k               |                 4.3k                 |
| **支持语言数** |               297                |                 249                  |
| **插件支持**   |                是                |                  否                  |
| **渲染性能**   |                中                |                  高                  |
| **模块化**     |                是                |                  否                  |

在此过程中自己也参考了其它的一些高亮方案，尤其是[这篇文章](https://frostming.com/2020/12-08/highlight-lexer-comparison)通过表格形式详细的对比常用的代码高亮框架，虽然有比`Prism`更强的实现，但`Prism`已经基本上满足自己的要求，同时自己在`GitBook`的搭建过程中也积累了一定的使用经验，故最终决定采用`Prism`替换`Chroma`。

## 操作步骤

由于修改时涉及到的工作量较大，考虑到自己的`Golang`和`Hugo`是半吊子水平，我决定基于已有的进行改进。

简单搜索一番后，在[这条回复](https://discourse.gohugo.io/t/is-there-a-proper-way-to-replace-chroma-with-prism-highlighting/38309/3)下找到了`Hugo`核心开发人员[Joe Mooring](https://github.com/jmooring)给出了其已经开发好的demo，基于下述指令可直接运行测试

```bash
git clone --single-branch -b hugo-forum-topic-38309 https://github.com/jmooring/hugo-testing hugo-forum-topic-38309
cd hugo-forum-topic-38309
hugo server
```

本地运行后发现确实能够正常使用，但与自己在`GitBook`中看到的效果有一定的差距，决定对其进行分步骤改造，相关操作步骤如下：

1. 在`layouts/_default/_markup`下新建文件`render-codeblock.html`，其内容如下，其内容相对于`hugo-forum-topic-38309`分支上的[原始文件](https://github.com/jmooring/hugo-testing/blob/hugo-forum-topic-38309/layouts/_default/_markup/render-codeblock.html)做了一定的改动，主要是添加自动显示行号相关的信息

   ```go
   <!-- only show line numbers when there is more than one line or specific it manually -->
   {{- $classStr := "language-%s" }}
   {{- $lineCount := len (split .Inner "\n") -}}
   {{- if gt $lineCount 1 }}
     {{- $classStr = print "line-numbers " print $classStr  }}
   {{- else -}}
     {{- $classStr = print "no-line-numbers " print $classStr  }}
   {{- end }}
   
   {{- $attributes := .Attributes }}
   {{- $classes := slice (printf $classStr .Type) .Attributes.class }}
   {{- $attributes = merge $attributes (dict "class" (delimit $classes " ")) }}
   <pre
     {{- range $k, $v := $attributes }}
       {{- printf " %s=%q" $k $v | safeHTMLAttr }}
     {{- end -}}
   >
   <code>
   {{- .Inner -}}
   </code>
   </pre>
   {{- .Page.Store.Set "hasCodeBlock" true }}
   ```

2. 在`assets/sass/_custom/_custom.scss`中添加如下内容，主要用于正确显示行号位置

   ```css
   .line-numbers .line-numbers-rows{
       border-right: 0px !important;
   }
   
   /*只有一行代码时没必要显示行号*/
   .no-line-numbers{
     position: relative
   }
   ```

3. 将`assets/sass/_partial/_post/_code.scss`的内容精简保留如下，其余的需要移除掉，否则由于样式冲突导致显示不正常 

   ```css
   code, pre {
     /* padding: 7px;
     font-size: $code-font-size;*/
     font-family: $code-font-family;
     background: $code-background;
   }
   
   code {
     /*padding: 3px 5px;*/
     border-radius: 4px;
     color: $code-color;
   }
   
   pre[class*=language-] {
      font-size: $code-font-size;
   }
   ```

4. 将`config.toml`中的下述内容移除掉

   ```toml
   [markup.highlight]
       anchorLineNos = false
       codeFences = true
       guessSyntax = true
       hl_Lines = ""
       lineAnchors = ""
       lineNoStart =1
       lineNos = true
       lineNumbersInTable = true
       noClasses = true
       style = "monokai"
       tabWidth = 4
   ```

5. 在`layouts/partials/head.html`中添加如下内容

   ```html
   {{- if .Store.Get "hasCodeBlock" -}}
   <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/themes/prism-solarizedlight.min.css"/>
   <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-highlight/prism-line-highlight.min.css"/>
   <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-numbers/prism-line-numbers.min.css"/>
   {{- end -}}
   ```

6. 在`layouts/partials/scripts.html`中添加如下内容

   ```html
   {{ if .Store.Get "hasCodeBlock" }}
     <script src="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/components/prism-core.min.js"></script>
     <script src="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/autoloader/prism-autoloader.min.js"></script>
     <script src="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-highlight/prism-line-highlight.min.js"></script>
     <script src="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-numbers/prism-line-numbers.min.js"></script>
   {{ end }}
   ```

   同时代码复制部分的源码修改如下，以便保留原有的代码复制按钮样式

   ```javascript
   <!-- copy to clipboard -->
   {{- if .Site.Params.enableCopyCode -}}
   <script>
     let copyIcon = `<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 16 16">
     <path fill-rule="evenodd" 
     d="M4 2a2 2 0 0 1 2-2h8a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2zm2-1a1 1 0 0 0-1 1v8a1 1 0 0 0 1 1h8a1 1 0 0 0 1-1V2a1 1 0 0 0-1-1zM2 
         5a1 1 0 0 0-1 1v8a1 1 0 0 0 1 1h8a1 1 0 0 0 1-1v-1h1v1a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h1v1z"/>
   </svg>`;
     let copiedIcon=`<svg clip-rule="evenodd" fill-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" width="16" height="16" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
     <path d="m2.25 12.321 7.27 6.491c.143.127.321.19.499.19.206 0 .41-.084.559-.249l11.23-12.501c.129-.143.192-.321.192-.5 0-.419-.338-.75-.749-.75-.206 0-.411.084-.559.249l-10.731 11.945-6.711-5.994c-.144-.127-.322-.19-.5-.19-.417 0-.75.336-.75.749 0 .206.084.412.25.56" fill-rule="nonzero"/></svg>`;
     function createCopyButton(codeDiv) {
       const div = document.createElement("div");
       div.className = "copy-code";
       div.innerHTML = copyIcon;
       div.addEventListener("click", () =>
         copyCodeToClipboard(div, codeDiv)
       );
       addCopyButtonToDom(div, codeDiv);
     }
   
     async function copyCodeToClipboard(button, codeDiv) {
       const codeToCopy = codeDiv.querySelector(":scope > code")
         .innerText;
       await navigator.clipboard.writeText(codeToCopy);
       button.blur();
   	button.innerHTML = copiedIcon;
       setTimeout(() => button.innerHTML = copyIcon, 2000);
     }
   
     function addCopyButtonToDom(button, codeDiv) {
       const wrapper = document.createElement("div");
       wrapper.className = "highlight-wrapper";
       codeDiv.parentNode.insertBefore(wrapper, codeDiv);
       wrapper.appendChild(codeDiv);
   	wrapper.insertBefore(button, wrapper.firstChild);
     }
   
     var isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
     if(!isMobile){
        document.querySelectorAll("pre[class*=language]").forEach((codeDiv) => createCopyButton(codeDiv));
     }
   </script>  
   {{ end }}
   ```

## 效果对比

基于前述步骤修改完毕后，可进行简单的测试，测试对比效果如下：

1. `Java`代码显示结果对比

   ![Java显示效果对比](/blog_img/hugo/using-prismjs-to-highlight-code/hugo_java_code_compare.png "Java显示效果对比")

2. `JavaScript`代码显示结果对比

   ![JavaScript显示效果对比](/blog_img/hugo/using-prismjs-to-highlight-code/hugo_js_code_compare.png "JavaScript显示效果对比")

   可看出整体上`Prismjs`的显示效果与识别度整体上比`Chroma`要更好。

## 注意事项

1. 由于`Prismjs`在数据库部分支持的语言为`sql`，对于细分的`mysql`、`sql server`等尚未提供支持，所以需要将涉及到的代码块语言进行统一修改

   ![替换不支持的语言](/blog_img/hugo/using-prismjs-to-highlight-code/replace-mysql-with-sql-in-code-block.png "替换不支持的语言")

2. 测试`Prismjs`非`Hugo`原生的，故在引入对应的`js`和`css`文件时，需要采用适合自己网络环境的CDN或本地文件，本例中采用的是基于[https://cdn.jsdelivr.net](https://cdn.jsdelivr.net)的网络文件

[^1]: https://gohugo.io/content-management/syntax-highlighting