---
title: "利用Html文件生成chm文件"
date: 2024-11-25T10:22:35+08:00
lastmod: 2024-11-25T10:22:35+08:00
draft: false
keywords: ["chm","html","目录","脚本"]
description: "对个人使用HTML网页生成CHM格式说明手册的使用经验进行简单的记录与分享，包含文档的生成，目录的配置以及踩坑事项。"
tags: ["web"]
categories: ["工具使用","文档管理"]
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

对个人使用`HTML`网页生成[**CHM**](https://en.wikipedia.org/wiki/Microsoft_Compiled_HTML_Help)格式说明手册的使用经验进行简单的记录与分享。

<!--more-->

## 背景

`CHM`(Microsoft **Compiled HTML Help**)是微软公司开发的文件帮助格式，包含了由一系列的`HTML`页面、目录菜单、索引和其它导航工具，在`Windows`环境下可方便的打开与查阅。

由于其使用的便利性(`Windows`环境下可直接打开，无须各种依赖)，故相同当多的软件在`Windows`环境下都提供了`CHM`格式的说明手册，出于此考虑个人想将部门的部分使用软件整合为`CHM`格式，在此过程中发现网络上关于此方面的资料较少，故简单记录下。

## 软件安装

{{% admonition info "说明" false %}}

由于`CHM`是微软开发的，其自家的操作系统为`Windows`，故本文的操作环境也为`Windows`。

{{% /admonition %}}

点击[此链接](https://www.helpandmanual.com/download/htmlhelp.exe)下载名为`htmlhelp.exe`的文件，双击安装完毕后其目录结构类似如下

![htmlhelp安装结果](/blog_img/web/using-html-files-to-create-chm-file/html-help-install-result.png "htmlhelp安装结果")

从图中可看出部分文件的日期相当老旧，事实上自从2009年起微软官方便计划不在提供新版本的`CHM`[^1]，尽管如此但并不影响`CHM`的正常使用。

将安装路径添加到系统环境变量后，可采用类似如下方式验证安装是否成功

![hhc命令检查](/blog_img/web/using-html-files-to-create-chm-file/hhc-command-check.png "hhc命令检查")

## CHM生成

环境准备好之后，可以通过下述步骤实现将1个简单的`HTML`文件生成`CHM`文件:

1.编写一个简单的`HTML`文件，假设其名称为`index.html`，内容如下

```html
<!DOCTYPE html>
<html>
    <head>
        <title>CHM测试</title>
		<meta charset="utf-8">
		<style type="text/css">
			.content {
			     background-color: powderblue;
				 width:80%;
				 margin-left:auto;
				 margin-right:auto;
				 padding: 10px;
				 min-height:100px;
			}
			
			.content:hover {
				 background-color: #3e8e41;
				 color: white;
            }
		</style>
    </head>
    <body>
        <p class="content">测试生成简单的<strong>CHM</strong>文件</p>
    </body>
</html>
```

该文件的显示效果如下

![index.html显示效果](/blog_img/web/using-html-files-to-create-chm-file/index-html-page.png "index.html显示效果")

2.编写一个扩展名为`hhp`(全称为`HTML Help Project`)的文件，文件名称不限制，内容如下

```ini
[OPTIONS]
Compatibility=1.1 or later
Compiled file=help.chm
Default topic=index.html
Display compile progress=Yes
Language=0x804 中文（简体，中国）

[FILES]
index.html
```

关于上述文件中的属性具体说明可参见[此链接](https://www.cnblogs.com/yunfeifei/p/14139956.html)或[此链接](https://www.cnblogs.com/lin1270/archive/2011/12/21/2295860.html)，也可查看老外写的[更加详细的说明](https://www.nongnu.org/chmspec/latest/INI.html)或[在线文档](https://www.alexthayer.com/UsingHHPFiles.pdf)。

3.假设前述步骤中的文件名为`chm-test.hhp`，在命令行执行如下命令

```bash
hhc chm-test.hhp
```

若环境配置正确，则输出类似如下

![chm生成过程输出](/blog_img/web/using-html-files-to-create-chm-file/chm-build-progress-1.png "chm生成过程输出")

4.若一切正常，则会在当前目录生成一个名为`help.chm`的文件，打开后其展示效果如下，可看出其展示效果和步骤1中的`HTML`页面直接展示类似

![简单的chm页面展示](/blog_img/web/using-html-files-to-create-chm-file/simple-chm-page-render.png "简单的chm页面展示")

## 中文文件名

若想将生成的`CHM`文件改为中文名称，可将前面的`hhp`文件内容修改如下

```ini
[OPTIONS]
Compatibility=1.1 or later
Compiled file=帮助手册.chm
Default topic=index.html
Display compile progress=Yes
Language=0x804 中文（简体，中国）

[FILES]
index.html
```

执行编译命令的输出类似如下，可看出输出过程中的中文显示乱码，实际生成的`CHM`文件名称也确实是乱码。

![中文文件名显示不正常](/blog_img/web/using-html-files-to-create-chm-file/chm-chinese-file-not-working.png "中文文件名显示不正常")

此问题一般是由编码问题造成的，需要将`hhp`文件的编码从`UTF-8`修改为`ANSI`

![将文件编码修改为ANSI](/blog_img/web/using-html-files-to-create-chm-file/change-file-encoding-to-ansi.png "将文件编码修改为ANSI")

重新执行构建命令后可发现能正常生成中文文件

![中文文件名显示正常](/blog_img/web/using-html-files-to-create-chm-file/chm-chinese-file-working.png "中文文件名显示正常")

## 中文标题支持

如下图所示，默认情况下生成的`CHM`文件标题在中文环境为**帮助**，在英文环境下为**Help**，看起来不太直观。

![chm文件默认标题](/blog_img/web/using-html-files-to-create-chm-file/chm-default-title.png "chm文件默认标题")

可通过[此说明](https://www.nongnu.org/chmspec/latest/INI.html)在`hhp`文件中设置`Title`属性来实现，根据说明可修改为如下

```ini
[OPTIONS]
Compatibility=1.1 or later
Compiled file=帮助手册.chm
Default topic=index.html
Display compile progress=Yes
Language=0x804 中文（简体，中国）
Title=简单的说明文档

[FILES]
index.html
```

此时如果执行构建指令，会发现`CHM`文件的标题还是没有变化，网络上这方面的资料较少，经过自己的一番对比与尝试后，**发现`Title`属性必须与其它功能结合后才生效**，如搜索功能、菜单目录等，原因暂时不明。

假设要添加搜索功能，将`hhp`文件修改如下

```ini
[OPTIONS]
Compatibility=1.1 or later
Compiled file=帮助手册.chm
Default topic=index.html
Display compile progress=Yes
Language=0x804 中文（简体，中国）
Title=简单的说明文档
;支持搜索功能
Full-text search=Yes

[FILES]
index.html
```

重新执行构建指令，并打开生成的`CHM`文件，可发现其标题已经改变。

![chm文件自定义标题](/blog_img/web/using-html-files-to-create-chm-file/chm-custom-title.png "chm文件自定义标题")

## 菜单目录支持

前述的内容都是基于单个`HTML`页面，实际使用中肯定会涉及多个文件，尤其是使用手册等说明类的文档，通常需要添加菜单目录来进行合适的分类与快速导航定位。

可通过在编译时提供`hhc`(全称为`HTML Help Contents`)文件来生成带有菜单目录的`CHM`文件，相关操作步骤如下：

1.为了便于管理，将相关的文件都放到一个文件夹下，其层级结构如下所示，其中包含多个`HTML`文件和图片等文件

![复杂的文档结构](/blog_img/web/using-html-files-to-create-chm-file/complex-docs-structure.png "复杂的文档结构")

2.建议一个如下图所示的`hhp`文件，可发现其与前面的`hhp`文件有一定差异，通过`Contents file`文件来配置菜单目录，同时无需在`FILES`下面指定具体的文件列表(因为在`hhc`文件中已经包含了相关的文件)

```ini
[OPTIONS]
Compatibility=1.1 or later
Compiled file=帮助手册.chm
;此处用于设置菜单目录
Contents file=contents.hhc
Default topic=doc/ch1/index.html
Display compile progress=Yes
Language=0x804 中文（简体，中国）
Title=简单的说明文档
```

3.对应的`contents.hhc`(文件名可任意起名)文件内容如下，可发现其本质上是`HTML`文件，其中的`UL`相关的嵌套代码即为最终要展示的层级结构

{{< details "+点击以展开/折叠" >}} 

```html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML//EN">
<HTML>
   <HEAD>
      <meta name="GENERATOR" content="Microsoft&reg; HTML Help Workshop 4.1">
      <!-- Sitemap 1.0 -->
   </HEAD>
   <BODY>
      <OBJECT type="text/site properties">
         <param name="ImageType" value="Folder">
      </OBJECT>
      <UL>
         <LI>
            <OBJECT type="text/sitemap">
               <param name="Name" value="第1章节">
               <param name="Local" value="doc/ch1/index.html">
               <param name="ImageNumber" value="1">
            </OBJECT>
            <UL>
               <LI>
                  <OBJECT type="text/sitemap">
                     <param name="Name" value="功能1">
                     <param name="ImageNumber" value="0">
                  </OBJECT>
                  <UL>
                     <LI>
                        <OBJECT type="text/sitemap">
                           <param name="Name" value="页面1">
                           <param name="Local" value="doc/ch1/func1/page1.html">
                        </OBJECT>
                     </LI>
                     <LI>
                        <OBJECT type="text/sitemap">
                           <param name="Name" value="页面2">
                           <param name="Local" value="doc/ch1/func1/page2.html">
                        </OBJECT>
                     </LI>
                  </UL>
               </LI>
               <LI>
                  <OBJECT type="text/sitemap">
                     <param name="Name" value="About">
                     <param name="Local" value="doc/ch1/about.html">
                  </OBJECT>
               </LI>
               <LI>
                  <OBJECT type="text/sitemap">
                     <param name="Name" value="Menu">
                     <param name="Local" value="doc/ch1/menu.html">
                  </OBJECT>
               </LI>
            </UL>
         </LI>
         <LI>
            <OBJECT type="text/sitemap">
               <param name="Name" value="第2章节">
               <param name="ImageNumber" value="1">
            </OBJECT>
            <UL>
               <LI>
                  <OBJECT type="text/sitemap">
                     <param name="Name" value="功能1">
                     <param name="ImageNumber" value="0">
                  </OBJECT>
               </LI>
               <LI>
                  <OBJECT type="text/sitemap">
                     <param name="Name" value="概览">
                     <param name="Local" value="doc/ch2/func1.html">
                  </OBJECT>
               </LI>
               <LI>
                  <OBJECT type="text/sitemap">
                     <param name="Name" value="具体说明">
                     <param name="Local" value="doc/ch2/具体说明.html">
                  </OBJECT>
               </LI>
            </UL>
         </LI>
      </UL>
   </BODY>
</HTML>
```

{{< /details>}}

4.构建过程如下，由于其包含的文件较多，故构建过程中输出的日志也更加详细

![复杂的文档构建](/blog_img/web/using-html-files-to-create-chm-file/chm-build-progress-2.png "复杂的文档构建")

5.打开生成的文件，展示效果类似如下，可看出左侧出现了我们基于`hhc`文件设置的菜单，且能够正常的点击切换

![复杂的文档展示效果](/blog_img/web/using-html-files-to-create-chm-file/complex-chm-render.png "复杂的文档展示效果")

## 文件图标修改

前述的菜单目录实现的核心为`<OBJECT>`对象，其中包含一个或多个`<param>`对象，通过键值对的方式设置相关参数值，可选的参数如下：

| 参数          | 说明                                                       |
| ------------- | ---------------------------------------------------------- |
| `Name`        | 用于设置在菜单目录中的显示名称(即文件名和显示名称可不相同) |
| `Local`       | 该对象对应的`HTML`文档，采用`相对路径`设置                 |
| `ImageNumber` | 该对象在菜单目录中使用何种图标                             |

其中`ImageNumber`的值可以设置为`从1到42`(若超过42不会报错且用默认值替代)，分别对应如下图标

![chm图标列表](/blog_img/web/using-html-files-to-create-chm-file/chm-icons-list.jpg "chm图标列表")

可通过如下代码来修改其值

```html
<OBJECT type="text/sitemap">
    <param name="Name" value="第1章节">
    <param name="Local" value="doc/ch1/index.html">
    <param name="ImageNumber" value="3">
</OBJECT>
```

基于上述说明修改前面的`hhc`文件，展示效果如下

![chm展示多个图标](/blog_img/web/using-html-files-to-create-chm-file/chm-multiple-icons.png "chm展示多个图标")

---

下述内容来源于[此文章](https://www.cnblogs.com/lin1270/archive/2011/12/21/2295860.html)，个人并未做实际验证

> 如果`<OBJECT>`中没有`ImageNumber`项时，如果`hhp`文件中的`ImageType`为`Picture`（默认是`Picture`），如果有子目录，就显示![图标1](/blog_img/web/using-html-files-to-create-chm-file/icon-1.jpg "图标1")，否则显示![图标2](/blog_img/web/using-html-files-to-create-chm-file/icon-2.jpg "图标2")，如果 `ImageType`为`Folder`，如果有子目录，就显示![图标3](/blog_img/web/using-html-files-to-create-chm-file/icon-3.jpg "图标3")，否则显示![图标4](/blog_img/web/using-html-files-to-create-chm-file/icon-4.jpg "图标4")

## 中文搜索支持

可通过在`hhp`文件中添加`Full-text search`来添加搜索功能

```ini
[OPTIONS]
Compatibility=1.1 or later
Compiled file=帮助手册.chm
;此处用于设置菜单目录
Contents file=contents.hhc
Default topic=doc/ch1/index.html
Display compile progress=Yes
Language=0x804 中文（简体，中国）
Title=简单的说明文档

;支持搜索功能
Full-text search=Yes
```

但个人在实际使用中遇到了中文搜索的一些坑，**需要将文件编码设置为`ANSI` 且在`HTML`文件中去掉显示的文件编码才能正常支持中文搜索**。

演示说明如下：

1.假设所有的`HTML`文件编码均为`UTF-8`且在文件源码中也通过如下代码进行显示声明

```html
<!--设置为utf-8编码 -->
<meta charset="utf-8">
```

2.在生成的`CHM`文件中搜索中文关键字，可看出虽然实际有内容，但检索结果为空

![中文搜索找不到结果](/blog_img/web/using-html-files-to-create-chm-file/chinese-search-not-found.png "中文搜索找不到结果")

3.切换为英文单词搜索后，结果如下，虽然有结果，但是显示为乱码，同样无法使用

![英文搜索结果乱码](/blog_img/web/using-html-files-to-create-chm-file/english-search-result-incorrect.png "英文搜索结果乱码")

4.将对应的`HTML`文件以`ANSI`编码格式保存，重新生成`CHM`文件，发现其中文直接显示乱码，此时进行搜索已经无意义

![CHM中文显示乱码](/blog_img/web/using-html-files-to-create-chm-file/chm-chinese-content-incorrect.png "CHM中文显示乱码")

5.基于前述步骤，将对应`HTML`文件源码中的编码类型去掉，然后重新生成`CHM`文件

```html
<!--设置为utf-8编码 
<meta charset="utf-8">-->
```

6.对重新生成的`CHM`文件进行中文检索，此时结果正常，如下图所示

![正常搜索结果展示](/blog_img/web/using-html-files-to-create-chm-file/chm-search-correct-result.png "正常搜索结果展示")

从上图的搜索结果可发现其展示的检索结果为`HTML`文件的`title`属性值而非实际的文件名称，故建议**在生成`hhc`文件时，`<OBJECT>`对象下的`Name`属性值要和`HTML`页面的`title`值保持一致**，避免使用上引起歧义，可通过代码脚本来达到此目的。

## 遗留问题

前述生成`CHM`文件的操作适用于简单的`HTML`文件，若`HTML`文件本身内容很复杂(如基于[GitBook](/tags/gitbook)生成)，则生成的`CHM`文件打开后可能会报错，原因尚未找出。

![复杂HTML文件显示错误](/blog_img/web/using-html-files-to-create-chm-file/complex-html-files-error.png "复杂HTML文件显示错误")

[^1]: [Microsoft HTML Help Downloads](https://learn.microsoft.com/zh-cn/previous-versions/windows/desktop/htmlhelp/microsoft-html-help-downloads?redirectedfrom=MSDN)

