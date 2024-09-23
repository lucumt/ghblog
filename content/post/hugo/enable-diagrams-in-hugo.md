---
title: "在Hugo中开启图表支持"
date: 2023-03-27T09:46:50+08:00
lastmod: 2023-03-27T09:46:50+08:00
draft: false
keywords: ["flowchart","sequence","mermaid","hugo","markdown"]
description: "简要记录如何在Hugo博客中集成flowchart、sequence和mermaid这3种图表，以增强表格的功能"
tags: ["hugo","go"]
categories: ["个人博客","系统集成"]
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

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: true
  options: "{
              'x': 0,
              'y': 0,
              'width':1,
              'line-width': 1,
              'line-length': 50,
              'text-margin': 10,
              'font-size': 14,
              'font-color': 'black',
              'line-color': 'black',
              'element-color': 'black',
              'fill': 'white',
              'yes-text': 'yes',
              'no-text': 'no',
              'arrow-end': 'block',
              'scale': 1,
              'symbols': {
                  'start': {
                    'font-color': 'red',
                    'element-color': 'green',
                    'fill': 'yellow'
                  },
                  'end': {
                      'class': 'end-element',
                      'element-color': 'green'
                  }
              },
              'flowstate': {
                'aaa': {'fill': 'pink'},
                'approved': {'fill': 'peru'}
              }
            }"

sequenceDiagrams: 
  enable: true
  options: "{
               'theme': 'simple',
               'width':1,
               'line-width': 1,
               'font-size': 14,
               'font-family': 'Andale Mono, monospace',
               'line':{
                   'stroke-width': 1
                },
               'rect':{
                   'stroke-width': 1,
                   'fill':'#deffcc'
                },
                'text':{
                    'fill':'#219b15',
                    'stroke-width':1,
                    'stroke':'#219b15'
                }
            }"

mermaidDiagrams: 
  enable: true
  options: "{
     'theme':'forest'
  }"
---

{{% admonition  info "说明" false %}}

若页面中图表显示不正常，可能是由于`js`文件加载和图表渲染需要时间，请多等待几秒钟！

{{% /admonition %}}



简要介绍在如何在`Hugo`博客中基于[Even](https://github.com/olOwOlo/hugo-theme-even)主题开启[flowchart](https://flowchart.js.org/)、[sequence](https://bramp.github.io/js-sequence-diagrams/)和[mermaid](https://mermaid.js.org/)图表的支持。

<!--more-->

## 背景

个人`Hugo`博客切换为`Even`已经有好几年了，相关功能也是基于此主题扩展而来，不过`Even`主题的作者已经很久没有此主题,GitHub上大量的issue都由于过期而自动关闭[^1]，同时个人发现该主题下`layouts/partials/scripts.html`对于`flowchart`和`sequence`的实现并不完善，缺少对应的初始化代码，基于此我决定在该主题的基础上自己实现相关功能！

PS: `Even`主题[^2]很受欢迎，希望作者早日恢复维护!

![Even主题的作者已经没有维护](/blog_img/hugo/enable-diagrams-in-hugo/even-theme-website-not-display-diagram.png "Even主题的作者已经没有维护") 

![Even主题无法初始化图表](/blog_img/hugo/enable-diagrams-in-hugo/even-theme-does-not-have-diagram-init-code.png "Even主题无法初始化图表") 

## 修改说明

由于`Hugo`支持代码高亮，当图表相关的数据当采用`Markdown`的codeblock方式写入时，如下图所示默认情况下，其会以代码高亮的方式显示，很明显此种方式不满足要求。

![默认情况下按照代码高亮显示](/blog_img/hugo/enable-diagrams-in-hugo/hugo-diagram-show-as-highlight-code.png "默认情况下按照代码高亮显示") 

从上图可知经过高亮处理后的图表代码已经与`html`标签混杂在一起，为了能正常展示图表数据，必须要有一种方式能够准确的获取原始的图表代码，此时需要借助`Hugo`的[Render Hooks](https://gohugo.io/templates/render-hooks/)功能，其主要功能是**让我们通过自定义的模板来覆盖`Hugo`默认的功能实现**。如下图所示经过Markup处理后的，会将原始的代码文件基于对应codeblock文件中的代码来展示，在对应的`html`中包含我们原始的代码块文件，采用`JavaScript`或`jQuery`都能快速的获取到对应代码，之后调用对应图表`JavaScript`库来进行渲染或初始化即可。

![经过render hooks处理后按html展示](/blog_img/hugo/enable-diagrams-in-hugo/hugo-diagram-show-as-highlight-html.png "经过render hooks处理后按html展示") 

根据`Hugo`官方[Render Hooks](https://gohugo.io/templates/render-hooks/)上的说明，需要在`layouts/_default/_markup`文件夹下建立对应的`html`文件，其命名规则为`render-xxx-[yyy].html`,其中xxx为一级类别，yyy为二级类别(可选的)，如要重写所有的代码展示可创建文件`render-codeblock.html`，要重写所有的图片展示可创建文件`render-image.html`,要重写所有的`mermaid`代码展示则需要添加上二级分类`render-codeblock-mermiad.html`。

```
layouts/
└── _default/
    └── _markup/
        ├── render-codeblock-bash.html
        ├── render-codeblock.html
        ├── render-heading.html
        ├── render-image.html
        ├── render-image.rss.xml
        └── render-link.html
```

{{% admonition  warning "注意" false %}}

当我们最终修改完毕后，若页面上没有正常展示相关的图表且浏览器控制台出现如下错误，需要在`layouts/partials/scripts.html`或`config.toml`中重新修改相关报错文件的`integrity`值，或在`static/lib`下重新下载相关的文件，确保其校验值一致，即可消除错误。

{{% /admonition %}}

![加载js和css文件时提示资源完整性校验不通过](/blog_img/hugo/enable-diagrams-in-hugo/invalid_integrity_check_for_js_and_css.png "加载js和css文件时提示资源完整性校验不通过") 

## flowchart图表

### 修改过程

* 在`layouts/_default/_markup`创建文件`render-codeblock-flow.html`并添加如下代码

  ```html
  <div id="flow_{{ .Ordinal }}">
    {{- .Inner | safeHTML }}
  </div>
  ```

* 在`layouts/partials/scripts.html`补充原有的代码，添加上初始化功能

  ```html
  <!-- flowchart -->
  {{- if and (or .Params.flowchartDiagrams.enable (and .Site.Params.flowchartDiagrams.enable (ne .Params.flowchartDiagrams.enable false))) (or .IsPage .IsHome) -}}
    {{- if .Site.Params.publicCDN.enable -}}
      {{ .Site.Params.publicCDN.flowchartDiagramsJS | safeHTML }}
    {{- else -}}
      <script src="{{ "lib/flowchartDiagrams/raphael-2.2.7.min.js" | relURL }}" integrity="sha256-67By+NpOtm9ka1R6xpUefeGOY8kWWHHRAKlvaTJ7ONI=" crossorigin="anonymous"></script>
      <script src="{{ "lib/flowchartDiagrams/flowchart-1.8.0.min.js" | relURL }}" integrity="sha256-zNGWjubXoY6rb5MnmpBNefO0RgoVYfle9p0tvOQM+6k=" crossorigin="anonymous"></script>
    {{- end -}}
  	<script>
  	/*{{- if .Params.flowchartDiagrams.options -}}
  	  window.flowchartDiagramsOptions = {{ .Params.flowchartDiagrams.options | safeJS }};
  	{{- else if .Site.Params.flowchartDiagrams.options -}}
  	  window.flowchartDiagramsOptions = {{ .Site.Params.flowchartDiagrams.options | safeJS }};
  	{{- end -}}*/
  	
  	<!-- below is newly added code -->
  	let flowPageOptions = {{ .Page.Params.flowchartDiagrams.options  }};
  	let flowSiteOptions = {{ .Site.Params.flowchartDiagrams.options  }};
  	flowPageOptions = !!flowPageOptions ? flowPageOptions : "{}"
  	flowSiteOptions = !!flowSiteOptions ? flowSiteOptions : "{}"
  	
  	flowPageOptions = eval("(" + flowPageOptions + ")")
  	flowSiteOptions = eval("(" + flowSiteOptions + ")")
  	// page options have high priority then site options
  	let flowOptions = {...flowSiteOptions, ...flowPageOptions}; 
  	$("[id^=flow_]").flowChart(flowOptions);
  	</script>
  {{- end -}}
  ```

  上述代码中通过`let flowOptions = {...flowSiteOptions, ...flowPageOptions};`来确保页面上的配置覆盖全局配置，优先级更高。

* 在对应的`markdown`页面头部开启`flowchart`的展示，可根据实际情况添加自定义配置，保存对应`markdown`文件后页面会自动刷新并展示对应效果[^3]。

  ```json
  flowchartDiagrams:
    enable: true
    options: "{
                'x': 0,
                'y': 0,
                'width':1,
                'line-width': 1,
                'line-length': 50,
                'text-margin': 10
              }"
  ```

### 自定义样式

 不同于`sequence`,`flowchart`的使用很灵活，其[官网](https://flowchart.js.org/)上虽然只有4个demo，但已经覆盖了大部分功能，本小节简要介绍如何对其中的`flowstate`进行配置从而让图表展示不同的样式。



为了实现自定义样式的展示，需要了解`flowchart`的语法,下图展示了其主要语法，更具体的可参见[^4]，实际使用中我们主要通过对`flowstate`来实现自定义样式显示。

![flowchart流程图语法展示](/blog_img/hugo/enable-diagrams-in-hugo/flowchart-grammer-explanation.png "flowchart流程图语法展示") 

下图展示了一个通过设置不同`flowstate`来设置流程图不同组件样式的示例。
![flowstate自定义样式设置](/blog_img/hugo/enable-diagrams-in-hugo/flowchart-flowstate-custom-config.png "flowstate自定义样式设置") 

### 展示效果

基于对应`markdown`页面的下述配置展示相关效果

```json
flowchartDiagrams:
  enable: true
  options: "{
              'x': 0,
              'y': 0,
              'width':1,
              'line-width': 1,
              'line-length': 50,
              'text-margin': 10,
              'font-size': 14,
              'font-color': 'black',
              'line-color': 'black',
              'element-color': 'black',
              'fill': 'white',
              'yes-text': 'yes',
              'no-text': 'no',
              'arrow-end': 'block',
              'scale': 1,
              'symbols': {
                  'start': {
                    'font-color': 'red',
                    'element-color': 'green',
                    'fill': 'yellow'
                  },
                  'end': {
                      'class': 'end-element',
                      'element-color': 'green'
                  }
              },
              'flowstate': {
                'aaa': {'fill': 'pink'},
                'approved': {'fill': 'peru'}
              }
            }"
```

#### 图表1

* 原始代码

  ```
  ​```flow
  st=>start: 开始框
  op=>operation: 处理框
  cond=>condition: 判断框(是或否?)
  sub1=>subroutine: 子流程
  io=>inputoutput: 输入输出框|approved
  e=>end: 结束框
  st->op->cond
  cond(yes)->io->e
  cond(no)->sub1(right)->op
  ​```
  ```

* 展示效果

  ```flow
  st=>start: 开始框
  op=>operation: 处理框
  cond=>condition: 判断框(是或否?)
  sub1=>subroutine: 子流程
  io=>inputoutput: 输入输出框|approved
  e=>end: 结束框
  st->op->cond
  cond(yes)->io->e
  cond(no)->sub1(right)->op
  ```

#### 图表2

* 原始代码

  ```
  ​```flow
  st=>start: 开始节点
  in=>inputoutput: 输入
  e=>end: 结束节点
  op=>operation: 操作节点
  cond=>condition: 条件节点
  sub=>subroutine: 子例程
  out=>inputoutput: 输出
  st(right)->in->op->cond
  cond(yes,right)->out->e
  cond(no)->sub
  ​```
  ```

* 展示效果

  ```flow
  st=>start: 开始节点
  in=>inputoutput: 输入
  e=>end: 结束节点
  op=>operation: 操作节点
  cond=>condition: 条件节点
  sub=>subroutine: 子例程
  out=>inputoutput: 输出
  st(right)->in->op->cond
  cond(yes,right)->out->e
  cond(no)->sub
  ```

#### 图表3

* 原始代码

  ```
  ​```flow
  st=>start: Start|past:>http://www.google.com[blank]
  e=>end: End:>http://www.google.com
  op1=>operation: My Operation|past
  op2=>operation: Stuff|aaa
  sub1=>subroutine: My Subroutine|invalid
  cond=>condition: Yes or No?|approved:>http://www.google.com
  c2=>condition: Good idea|rejected
  io=>inputoutput: catch something...|aaa
  
  st->op1(right)->cond
  cond(yes, right)->c2
  cond(no)->sub1(left)->op1
  c2(yes)->io->e
  c2(no)->op2->e
  ​```
  ```

* 展示效果

  ```flow
  st=>start: Start|past:>http://www.google.com[blank]
  e=>end: End:>http://www.google.com
  op1=>operation: My Operation|past
  op2=>operation: Stuff|aaa
  sub1=>subroutine: My Subroutine|invalid
  cond=>condition: Yes
  or No?|approved:>http://www.google.com
  c2=>condition: Good idea | rejected
  io=>inputoutput: catch something...|aaa
  
  st->op1(right)->cond
  cond(yes, right)->c2
  cond(no)->sub1(left)->op1
  c2(yes)->io->e
  c2(no)->op2->e
  ```

## sequence图表

### 修改过程

* 在`layouts/_default/_markup`创建文件`render-codeblock-sequence.html`并添加如下代码

  ```html
  <div id="sequence_{{ .Ordinal }}">
      {{- .Inner | safeHTML }}
  </div>
  ```

* 在`layouts/partials/scripts.html`补充原有的代码，添加上初始化功能

  ```html
  <!-- js-sequence-diagrams -->
  {{- if and (or .Params.sequenceDiagrams.enable (and .Site.Params.sequenceDiagrams.enable (ne .Params.sequenceDiagrams.enable false))) (or .IsPage .IsHome) -}}
    {{- if .Site.Params.publicCDN.enable -}}
      {{ .Site.Params.publicCDN.sequenceDiagramsJS | safeHTML }}
      {{ .Site.Params.publicCDN.sequenceDiagramsCSS | safeHTML }}
    {{- else -}}
      <script src="{{ "lib/js-sequence-diagrams/webfontloader-1.6.28.js" | relURL }}" integrity="sha256-4O4pS1SH31ZqrSO2A/2QJTVjTPqVe+jnYgOWUVr7EEc=" crossorigin="anonymous"></script>
      <script src="{{ "lib/js-sequence-diagrams/snap.svg-0.5.1.min.js" | relURL }}" integrity="sha256-oI+elz+sIm+jpn8F/qEspKoKveTc5uKeFHNNVexe6d8=" crossorigin="anonymous"></script>
      <script src="{{ "lib/js-sequence-diagrams/underscore-1.8.3.min.js" | relURL }}" integrity="sha256-obZACiHd7gkOk9iIL/pimWMTJ4W/pBsKu+oZnSeBIek=" crossorigin="anonymous"></script>
      <script src="{{ "lib/js-sequence-diagrams/sequence-diagram.js" | relURL }}"></script>
  	<!--<script src="{{ "lib/js-sequence-diagrams/sequence-diagram-2.0.1.min.js" | relURL }}" integrity="sha384-8748Vn52gHJYJI0XEuPB2QlPVNUkJlJn9tHqKec6J3q2r9l8fvRxrgn/E5ZHV0sP" crossorigin="anonymous"></script>-->
  	<link rel="stylesheet" href="{{ "lib/js-sequence-diagrams/sequence-diagram-2.0.1.min.css" | relURL }}" integrity="sha384-6QbLKJMz5dS3adWSeINZe74uSydBGFbnzaAYmp+tKyq60S7H2p6V7g1TysM5lAaF" crossorigin="anonymous">
    {{- end -}}
    <script>
      /*{{- if .Params.sequenceDiagrams.options -}}
        window.sequenceDiagramsOptions = {{ .Params.sequenceDiagrams.options | safeJS }};
      {{- else if .Site.Params.sequenceDiagrams.options -}}
        window.sequenceDiagramsOptions = {{ .Site.Params.sequenceDiagrams.options | safeJS }};
      {{- end -}}*/
  	<!-- below is newly added code -->
      let seqPageOptions = {{ .Page.Params.sequenceDiagrams.options }};
  	let seqSiteOptions = {{ .Site.Params.sequenceDiagrams.options }};
  	seqPageOptions = !!seqPageOptions ? seqPageOptions : "{}"
  	seqSiteOptions = !!seqSiteOptions ? seqSiteOptions : "{}"
  	
  	seqPageOptions = eval("(" + seqPageOptions + ")")
  	seqSiteOptions = eval("(" + seqSiteOptions + ")")
  	// page options have high priority then site options
  	let seqOptions = {...seqSiteOptions, ...seqPageOptions}; 
  	$("[id^=sequence_]").sequenceDiagram(seqOptions);
    </script>
  {{- end }}
  ```

  可以看出，其实现代码与`flowchart`的类似。

* 在对应的`markdown`页面头部开启`sequence`的展示，可根据实际情况添加自定义配置，保存对应`markdown`文件后页面会自动刷新并展示对应效果。

  ```json
  sequenceDiagrams: 
    enable: true
    options: "{
                 'theme': 'simple',
                 'font-size': 14,
                 'font-family': 'Andale Mono, monospace'
              }"
  ```

### 自定义样式

在`sequence`的[官方网站](https://bramp.github.io/js-sequence-diagrams/)上对于该图表的初始化只提供的`theme`这一个属性而且其值也只有simple和hand两个选项，对于字体，背景色等没有像`flowchart`那么丰富的支持。

经过多次尝试后发现支持`theme`、`fonts-size`和`font-family`这3个属性配置，显然不能满足使用要求。

一开始自己以为是自己没找到地方，翻遍其[GitHub项目](https://github.com/bramp/js-sequence-diagrams)后在[sequence-diagram.js](https://github.com/bramp/js-sequence-diagrams/blob/master/dist/sequence-diagram.js)上找到了如下说明，**作者自己承认实现不够完善，但此项目已经至少3年没有维护！**

![sequence不能动态配置的说明](/blog_img/hugo/enable-diagrams-in-hugo/seqence-bug-for-dynamic-config.png "sequence不能动态配置的说明") 

What，于是乎我只能自己fork源码自己修改了，修改好的代码参见[sequence-diagram.js](https://github.com/lucumt/js-sequence-diagrams/blob/master/src/sequence-diagram.js),在使用时用此文件或者压缩后的替换原有的，然后修改`integrity`即可。

`sequence`图表是基于[SVG](https://www.w3schools.com/graphics/svg_intro.asp)实现，故而自己的修改也是从`SVG`着手，由于时间关系自己只修改了[SVG Line](https://www.w3schools.com/graphics/svg_line.asp)和[SVG Rectangle](https://www.w3schools.com/graphics/svg_rect.asp)两个组件，修改后的使用效果如下：

![sequence图表动态配置](/blog_img/hugo/enable-diagrams-in-hugo/sequence-custom-config.png "sequence图表动态配置") 

### 展示效果

基于对应`markdown`页面的下述配置展示相关效果

```json
sequenceDiagrams: 
  enable: true
  options: "{
               'theme': 'simple',
               'width':1,
               'line-width': 1,
               'font-size': 14,
               'font-family': 'Andale Mono, monospace',
               'line':{
                   'stroke-width': 1
                },
               'rect':{
                   'stroke-width': 1,
                   'fill':'#deffcc'
                },
                'text':{
                    'fill':'#219b15',
                    'stroke-width':1,
                    'stroke':'#219b15'
                }
            }"
```

#### 图表1

* 原始代码

  ```
  ​```sequence
  Title: Here is a title
  A->B: Normal line
  B-->C: Dashed line
  C->>D: Open arrow
  D-->>A: Dashed open arrow
  ​```
  ```

* 展示效果

  ```sequence
  Title: Here is a title
  A->B: Normal line
  B-->C: Dashed line
  C->>D: Open arrow
  D-->>A: Dashed open arrow
  ```

#### 图表2

* 原始代码

  ```
  ​```sequence
  # Example of a comment.
  Note left of A: Note to the\n left of A
  Note right of A: Note to the\n right of A
  Note over A: Note over A
  Note over A,B: Note over both A and B
  ​```
  ```

* 展示效果

  ```sequence
  # Example of a comment.
  Note left of A: Note to the\n left of A
  Note right of A: Note to the\n right of A
  Note over A: Note over A
  Note over A,B: Note over both A and B
  ```

#### 图表3

* 原始代码

  ```
  ​```sequence
  participant C
  participant B
  participant A
  Note right of A: By listing the participants\n you can change their order
  ​```
  ```

* 展示效果

  ```sequence
  participant C
  participant B
  participant A
  Note right of A: By listing the participants\n you can change their order
  ```

#### 图表4

* 原始代码

  ```
  ​```sequence
  Andrew->China: Says Hello
  Note right of China: China thinks\nabout it
  China-->Andrew: How are you?
  Andrew->>China: I am good thanks!
  ​```
  ```

* 展示效果

  ```sequence
  Andrew->China: Says Hello
  Note right of China: China thinks\nabout it
  China-->Andrew: How are you?
  Andrew->>China: I am good thanks!
  ```

#### 图表5

* 原始代码

  ```
  ​```sequence
  participant System
  participant App
  System->>App: Do you hear me
  App-->>Module: Alive?
  Module-->>App: Yay!
  App->>System: Stop
  ​```
  ```

* 展示效果

  ```sequence
  participant System
  participant App
  System->>App: Do you hear me
  App-->>Module: Alive?
  Module-->>App: Yay!
  App->>System: Stop
  ```

## mermaid图表

[mermaid](https://mermaid.js.org/)是一个功能强大的`Markdown`图表显示控件，其本身的功能已经包含前述的`flowchart`和`sequence`，但由于`Even`主题的作者默认并没有加上此图表的支持，同时`Hugo`的官网有专门的配置说明[^5]，故本次一并加上。

### 修改过程

* 在`layouts/_default/_markup`创建文件`render-codeblock-mermaid.html`并添加如下代码

  ```html
  <div class="mermaid">
    {{- .Inner | safeHTML }}
  </div>
  {{ .Page.Store.Set "hasMermaid" true }}
  ```

* 在`layouts/partials/scripts.html`添加如下代码

  ```html
  <!-- mermaid js -->
  {{- if and (or .Params.mermaidDiagrams.enable (and .Site.Params.mermaidDiagrams.enable (ne .Params.mermaidDiagrams.enable false))) (or .IsPage .IsHome) -}}
  	{{ if .Page.Store.Get "hasMermaid" }}
  	  <script type="module">
  		import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10.0.2/+esm'
  		//import mermaid from '{{ "lib/mermaid/mermaid.esm.min.mjs" | relURL }}'
  		
  		let mermaidPageOptions = {{ .Page.Params.mermaidDiagrams.options }};
  		let mermaidSiteOptions = {{ .Site.Params.mermaidDiagrams.options }};
  		mermaidPageOptions = !!mermaidPageOptions ? mermaidPageOptions : "{}"
  		mermaidSiteOptions = !!mermaidSiteOptions ? mermaidSiteOptions : "{}"
  
  		mermaidPageOptions = eval("(" + mermaidPageOptions + ")")
  		mermaidSiteOptions = eval("(" + mermaidSiteOptions + ")")
  		// page options have high priority then site options
  		let mermaidOptions = {...mermaidSiteOptions, ...mermaidPageOptions}; 
  
  		mermaid.initialize(mermaidOptions);
  	  </script>
  	{{ end }}
  {{- end }}
  ```

* 在对应的`markdown`页面头部开启`mermaid`的展示，可根据实际情况添加自定义配置，保存对应`markdown`文件后页面会自动刷新并展示对应效果

  ```json
  mermaidDiagrams: 
    enable: true
    options: "{
       'theme':'forest'
    }"
  ```

### 自定义样式

`mermaid`图表的自定义配置主要基于themes来实现，在其[官网文档](https://mermaid.js.org/config/theming.html)上有很详细的说明，下图为一个简单的示例

![mermaid图表动态配置](/blog_img/hugo/enable-diagrams-in-hugo/mermaid-custom-config.png "mermaid图表动态配置") 

### 展示效果

基于对应`markdown`页面的下述配置展示相关效果

```json
mermaidDiagrams: 
  enable: true
  options: "{
     'theme':'forest'
  }"
```

#### 图表1-Sequence

* 原始代码

  ```
  ​```mermaid
  sequenceDiagram
  Alice->>John: Hello John, how are you?
  loop Healthcheck
      John->>John: Fight against hypochondria
  end
  Note right of John: Rational thoughts!
  John-->>Alice: Great!
  John->>Bob: How about you?
  Bob-->>John: Jolly good!
  ​```
  ```

* 展示效果

  ```mermaid
  sequenceDiagram
  Alice->>John: Hello John, how are you?
  loop Healthcheck
      John->>John: Fight against hypochondria
  end
  Note right of John: Rational thoughts!
  John-->>Alice: Great!
  John->>Bob: How about you?
  Bob-->>John: Jolly good!
  ```

#### 图表2-Flow

* 原始代码

  ```
  ​```mermaid
  flowchart TD
      A[Christmas] -->|Get money| B(Go shopping)
      B --> C{Let me think}
      C -->|One| D[Laptop]
      C -->|Two| E[iPhone]
      C -->|Three| F[fa:fa-car Car]
  ​```
  ```

* 展示效果

  ```mermaid
  flowchart TD
      A[Christmas] -->|Get money| B(Go shopping)
      B --> C{Let me think}
      C -->|One| D[Laptop]
      C -->|Two| E[iPhone]
      C -->|Three| F[fa:fa-car Car]
  ```

#### 图表3-Class

* 原始代码

  ```
  ​```mermiad
  classDiagram
      Animal <|-- Duck
      Animal <|-- Fish
      Animal <|-- Zebra
      Animal : +int age
      Animal : +String gender
      Animal: +isMammal()
      Animal: +mate()
      class Duck{
        +String beakColor
        +swim()
        +quack()
      }
      class Fish{
        -int sizeInFeet
        -canEat()
      }
      class Zebra{
        +bool is_wild
        +run()
      }
  ​```
  ```

* 展示效果

  ```mermaid
  classDiagram
      Animal <|-- Duck
      Animal <|-- Fish
      Animal <|-- Zebra
      Animal : +int age
      Animal : +String gender
      Animal: +isMammal()
      Animal: +mate()
      class Duck{
        +String beakColor
        +swim()
        +quack()
      }
      class Fish{
        -int sizeInFeet
        -canEat()
      }
      class Zebra{
        +bool is_wild
        +run()
      }
  ```

#### 图表4-State

* 原始代码

  ```
  ​```mermaid
  stateDiagram-v2
      [*] --> Still
      Still --> [*]
      Still --> Moving
      Moving --> Still
      Moving --> Crash
      Crash --> [*]
  ​```
  ```

* 显示效果

  ```mermaid
  stateDiagram-v2
      [*] --> Still
      Still --> [*]
      Still --> Moving
      Moving --> Still
      Moving --> Crash
      Crash --> [*]
  ```

#### 图表5-ER

* 原始代码

  ```
  ​```mermaid
  erDiagram
      CUSTOMER }|..|{ DELIVERY-ADDRESS : has
      CUSTOMER ||--o{ ORDER : places
      CUSTOMER ||--o{ INVOICE : "liable for"
      DELIVERY-ADDRESS ||--o{ ORDER : receives
      INVOICE ||--|{ ORDER : covers
      ORDER ||--|{ ORDER-ITEM : includes
      PRODUCT-CATEGORY ||--|{ PRODUCT : contains
      PRODUCT ||--o{ ORDER-ITEM : "ordered in"
  ​```
  ```

* 展示效果

  ```mermaid
  erDiagram
      CUSTOMER }|..|{ DELIVERY-ADDRESS : has
      CUSTOMER ||--o{ ORDER : places
      CUSTOMER ||--o{ INVOICE : "liable for"
      DELIVERY-ADDRESS ||--o{ ORDER : receives
      INVOICE ||--|{ ORDER : covers
      ORDER ||--|{ ORDER-ITEM : includes
      PRODUCT-CATEGORY ||--|{ PRODUCT : contains
      PRODUCT ||--o{ ORDER-ITEM : "ordered in"
  ```

#### 图表6-Gantt

* 原始代码

  ```
  ​```mermaid
  gantt
      section Section
      Completed :done,    des1, 2014-01-06,2014-01-08
      Active        :active,  des2, 2014-01-07, 3d
      Parallel 1   :         des3, after des1, 1d
      Parallel 2   :         des4, after des1, d
      Parallel 3   :         des5, after des3, 3d
      Parallel 4   :         des6, after des2, 1d
  ​```
  ```

* 展示效果

  ```mermaid
  gantt
      section Section
      Completed :done,    des1, 2014-01-06,2014-01-08
      Active        :active,  des2, 2014-01-07, 3d
      Parallel 1   :         des3, after des1, 1d
      Parallel 2   :         des4, after des1, d
      Parallel 3   :         des5, after des3, 3d
      Parallel 4   :         des6, after des2, 1d
  ```

#### 图表7-UserJourney

* 原始代码

  ```
  ​```mermaid
  journey
      title My working day
      section Go to work
        Make tea: 5: Me
        Go upstairs: 3: Me
        Do work: 1: Me, Cat
      section Go home
        Go downstairs: 5: Me
        Sit down: 3: Me
  ​```
  ```

* 展示效果

  ```mermaid
  journey
      title My working day
      section Go to work
        Make tea: 5: Me
        Go upstairs: 3: Me
        Do work: 1: Me, Cat
      section Go home
        Go downstairs: 5: Me
        Sit down: 3: Me
  ```

#### 图表8-Git

* 原始代码

  ```
  ​```mermaid
  gitGraph
      commit
      commit
      branch develop
      checkout develop
      commit
      commit
      checkout main
      merge develop
      commit
      commit
  ​```
  ```

* 展示效果

  ```mermaid
  gitGraph
      commit
      commit
      branch develop
      checkout develop
      commit
      commit
      checkout main
      merge develop
      commit
      commit
  ```

#### 图表9-Pie

* 原始代码

  ```
  ​```mermaid
  pie title Pets adopted by volunteers
      "Dogs" : 386
      "Cats" : 85
      "Rats" : 15
  ​```
  ```

* 展示效果

  ```mermaid
  pie title Pets adopted by volunteers
      "Dogs" : 386
      "Cats" : 85
      "Rats" : 15
  ```

#### 图表10-Mindmap

* 原始代码

  ```
  ​```mermaid
  mindmap
    root((mindmap))
      Origins
        Long history
        ::icon(fa fa-book)
        Popularisation
          British popular psychology author Tony Buzan
      Research
        On effectivness<br/>and features
        On Automatic creation
          Uses
              Creative techniques
              Strategic planning
              Argument mapping
      Tools
        Pen and paper
        Mermaid
  ​```
  ```

* 展示效果

  ```mermaid
  mindmap
    root((mindmap))
      Origins
        Long history
        ::icon(fa fa-book)
        Popularisation
          British popular psychology author Tony Buzan
      Research
        On effectivness<br/>and features
        On Automatic creation
          Uses
              Creative techniques
              Strategic planning
              Argument mapping
      Tools
        Pen and paper
        Mermaid
  ```

#### 图表11-timeline

* 原始代码

  ```
  ​```mermaid
          timeline
          title 我的日常
          section 努力搬砖
            上午 : 早会: 收邮件/回复邮件 : 查看线上问题
            下午 : 需求评审会 : 小组周会: coding
            晚上: 加班coding
  ​```
  ```

* 展示效果

  ```mermaid
          timeline
          title 我的日常
          section 努力搬砖
            上午 : 早会: 收邮件/回复邮件 : 查看线上问题
            下午 : 需求评审会 : 小组周会: coding
            晚上: 加班coding
  ```

#### 图表12-graph

* 原始代码

  ```
  ​```mermaid
  graph TB
      c1-->a2
      subgraph one
      a1-->a2
      end
      subgraph two
      b1-->b2
      end
      subgraph three
      c1-->c2
      end
  ​```
  ```

* 展示效果

  ```mermaid
  graph TB
      c1-->a2
      subgraph one
      a1-->a2
      end
      subgraph two
      b1-->b2
      end
      subgraph three
      c1-->c2
      end
  ```

参考文章:

1. https://snowdreams1006.github.io/write/mermaid-flow-chart.html

[^1]:https://github.com/olOwOlo/hugo-theme-even/issues?q=is%3Aissue+is%3Aclosed
[^2]:作者个人网站地址为[https://olowolo.com](https://olowolo.com)，GitHub地址为[https://github.com/olOwOlo/hugo-theme-even](https://github.com/olOwOlo/hugo-theme-even)
[^3]:此处假设我们采用`hugo server -w -D`来开启草稿模式和动态监测模式
[^4]:https://github.com/adrai/flowchart.js
[^5]:https://gohugo.io/content-management/diagrams/#mermaid-diagrams