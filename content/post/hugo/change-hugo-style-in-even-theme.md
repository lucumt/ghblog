---
title: "在Hugo生成的博客中动态的修改样式"
date: 2022-08-01T17:04:09+08:00
lastmod: 2022-08-01T17:04:09+08:00
draft: false
keywords: ["hugo"]
description: "简要记录自己基于hugo中的Even主题对自己的博客显示样式进行修改，同时支持动态切换皮肤"
tags: ["hugo","Go"]
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
---

自己一直特别羡慕[**博客园**](https://www.cnblogs.com)上某些博主的博文样式（如[武培轩](https://www.cnblogs.com/wupeixuan/p/13450815.html),[JavaGuide](https://www.cnblogs.com/javaguide/p/16385150.html))，这些博文样式第一眼看起来就很清爽,让人很有阅读的欲望，而我总感觉自己博客的样式特丑陋。即使不关注内容，只看排版布局和样式，有时候自己写完一篇文章自己都不想去看，何况别人！



终于有一天我受不了自己的博客样式，决定对其升级一版，在这个过程中发现个人采用的博客主题[**Even**](https://github.com/olOwOlo/hugo-theme-even)功能很强大，基于上没做啥修改就达到了自己想要的效果，特此记录下。

<!--more-->

## 风格对比

个人博客基于[**Golang**](https://go.dev/)语言开发的[**Hugo**](https://gohugo.io/)最开始是采用[**hugo-redlounge**](https://github.com/tmaiaroto/hugo-redlounge/)主题来实现的，经过一段时间的使用之后感觉`hugo-redlounge`主题功能不丰富且左侧会固定占用一部分宽度来展示个人信息，而这部分其实并无太大意义，故后来将其切换为如今的`Even`样式。

* `hugo-redlounge`主题的博客：

  ![个人旧版博客](/blog_img/hugo/change-hugo-style-in-even-theme/personal_blog_old_style.png "个人旧版博客") 

* `Even`主题的博客

  ![Even主题博客](/blog_img/hugo/change-hugo-style-in-even-theme/personal_blog_even_style.png "Even主题博客") 

* `Even`主题的代码和引用样式

  ![Even主题中的代码和引用样式](/blog_img/hugo/change-hugo-style-in-even-theme/code-blockquote-style-in-even-theme.png "Even主题中的代码和引用样式") 

切换后虽然相对于与`hugo-redlounge`主题当前博客的功能和界面美观性有一定提升，但是和博客园中的博文样式比起来直观感觉还是有一定差距，尤其是在包含代码段时淡黄色的背景看起来略微不舒服(如上图)。

## 源码分析

![Even主题中的代码样式的设置](/blog_img/hugo/change-hugo-style-in-even-theme/even-theme-code-style-css-code.png "Even主题中的代码样式的设置") 

个人对于`Even`主题最不满意的是在显示代码段时其样式不太符合自己预期，在浏览器中分析代码段对应的`CSS`样式，可看出是由如下`CSS`代码控制的：

```css
.post .post-content code, .post .post-content pre {
    padding: 7px;
    font-size: .9em;
    font-family: Consolas,Monaco,Menlo,dejavu sans mono,bitstream vera sans mono,courier new,monospace;
    background: #f8f5ec;
}
```

分析`Even`主题的源码后发现其主要是基于[**Sass**](https://sass-lang.com/)框架通过[_code.scss](https://github.com/lucumt/ghblog/blob/master/assets/sass/_partial/_post/_code.scss)和[_content.scss](https://github.com/lucumt/ghblog/blob/master/assets/sass/_partial/_post/_content.scss)联合实现的，其中背景设置是通过`background: $code-background;`实现

![Even主题中通过scss设置代码块样式](/blog_img/hugo/change-hugo-style-in-even-theme/hugo-code-block-scss-source-code.png "Even主题中通过scss设置代码块样式") 

由于`Sass`支持通过变量来动态的设置`CSS`属性值，故`$code-background`显然也是一个变量，查看源码后发现其在[_variables.scss](https://github.com/lucumt/ghblog/blob/master/assets/sass/_variables.scss)中有如下定义:

```scss
// Color of the code background.
$code-background: $deputy-color !default;
```

在[_variables.scss](https://github.com/lucumt/ghblog/blob/master/assets/sass/_variables.scss)中继续查找`$deputy-color`的定义，可找出在该文件开头有如下代码，其中`$deputy-color`是通过`$theme-color-config`在`$theme-color-map`中查找相关的颜色设置，自己博客之前默认淡黄色的代码块样式就是通过`Default`样式设置的。

```scss
// ==============================
// Variables
// ==============================

// ========== Theme Color ========== //
// Config here to change theme color
// Default | Mint Green | Cobalt Blue | Hot Pink | Dark Violet
$theme-color-config: 'Mint Green';

// Default theme color map
$theme-color-map: (
  'Default': #c05b4d #f8f5ec,
  'Mint Green': #16982B #f5f5f5,
  'Cobalt Blue': #0047AB #f0f2f5,
  'Hot Pink': #FF69B4 #f8f5f5,
  'Dark Violet': #9932CC #f5f4fa
);

// Check theme color config.
// if it does not exist, use default theme color.
@if not(map-has-key($theme-color-map, $theme-color-config)) {
  $theme-color-config: 'Default';
}
$theme-color-list: map-get($theme-color-map, $theme-color-config);

// Default theme color of the site.
$theme-color: nth($theme-color-list, 1) !default;

// Deputy theme color of the site.
$deputy-color: nth($theme-color-list, 2) !default;
```

进一步分析后发现`$deputy-color`在[_variables.scss](https://github.com/lucumt/ghblog/blob/master/assets/sass/_variables.scss)中的多个地方都有使用，从而可以确定通过修改`$theme-color-config`的值就能达到动态更改博客主题样式的目的。

```scss
// Deputy theme color of the site.
$deputy-color: nth($theme-color-list, 2) !default;

// Backgroud color of the post toc.
$post-toc-backgroud: rgba($deputy-color, 0.6) !default;

// Border color of the table.
$content-table-border-color: darken($deputy-color, 3%) !default;

// Color of the code background.
$code-background: $deputy-color !default;
```

## 样式修改

由于`SCSS`文件需要编译成`CSS`文件后才能被使用，而`Hugo`默认版本是不支持`SCSS`编译的，故需要下载`Hugo Extended`版本

![Hugo下载Extended版本](/blog_img/hugo/change-hugo-style-in-even-theme/hugo-extended-version-download.png "Hugo下载Extended版本") 

下载完毕后，需将其添加到环境变量，其各种指令的用法与默认版`Hugo`的用法相同，根据实际情况在[_variables.scss](https://github.com/lucumt/ghblog/blob/master/assets/sass/_variables.scss)中修改`$theme-color-config`的值，然后调用`hugo server -w -D`等命令即可实时展示修改效果。

![Even主题颜色设置](/blog_img/hugo/change-hugo-style-in-even-theme/even-theme-color-config.png "Even主题颜色设置") 

### 结果展示

#### Default样式

![Even主题Default样式](/blog_img/hugo/change-hugo-style-in-even-theme/even-theme-default-style.png "Even主题Default样式") 

#### Mint Green样式

![Even主题Mint Green样式](/blog_img/hugo/change-hugo-style-in-even-theme/even-theme-mint-green-style.png "Even主题Mint Green样式") 

#### Cobalt Blue样式

![Even主题Cobalt Blue样式](/blog_img/hugo/change-hugo-style-in-even-theme/even-theme-cobalt-blue-style.png "Even主题Cobalt Blue样式") 

#### Hot Pink样式

![Even主题Hot Pink样式](/blog_img/hugo/change-hugo-style-in-even-theme/even-theme-hot-pink-style.png "Even主题Hot Pink样式") 

#### Dark Violet样式

![Even主题Dark Violet样式](/blog_img/hugo/change-hugo-style-in-even-theme/even-theme-dark-violet-style.png "Even主题Dark Violet样式") 

### 扩展

可根据个人喜好在[_variables.scss](https://github.com/lucumt/ghblog/blob/master/assets/sass/_variables.scss)动态的添加自己喜欢的颜色配置

```scss
// ========== Theme Color ========== //
// Config here to change theme color
// Default | Mint Green | Cobalt Blue | Hot Pink | Dark Violet
$theme-color-config: 'Mint Green';

// Default theme color map
$theme-color-map: (
  'Default': #c05b4d #f8f5ec,
  'Mint Green': #16982B #f5f5f5,
  'Cobalt Blue': #0047AB #f0f2f5,
  'Hot Pink': #FF69B4 #f8f5f5,
  'Dark Violet': #9932CC #f5f4fa
);
```

## 致谢

* https://github.com/olOwOlo/hugo-theme-even
* https://github.com/ahonn/hexo-theme-even