---
title: "在Hugo中开启图表支持"
date: 2023-03-27T09:46:50+08:00
lastmod: 2023-03-27T09:46:50+08:00
draft: false
keywords: ["flowchart","sequence","mermaid","hugo","markdown"]
description: ""
tags: ["hugo","go"]
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

简要介绍在如何在`Hugo`博客中基于[Even](https://github.com/olOwOlo/hugo-theme-even)主题开启[flowchart](https://flowchart.js.org/)、[sequence](https://bramp.github.io/js-sequence-diagrams/)和[mermaid](https://mermaid.js.org/)图表的支持。

<!--more-->

# 背景

个人`Hugo`博客切换为`Even`已经有好几年了，相关功能也是基于此主题扩展而来，不过`Even`主题的作者已经很久没有此主题,GitHub上大量的issue都由于过期而自动关闭[^1]，同时个人发现该主题下`layouts/partials/scripts.html`对于`flowchart`和`sequence`的实现并不完善，缺少对应的初始化代码，基于此我决定在该主题的基础上自己实现相关功能！

PS: `Even`主题很受欢迎，希望作者早日恢复维护!

![Even主题的作者已经没有维护](/blog_img/hugo/enable-diagrams-in-hugo/even-theme-website-not-display-diagram.png "Even主题的作者已经没有维护") 

![Even主题无法初始化图表](/blog_img/hugo/enable-diagrams-in-hugo/even-theme-does-not-have-diagram-init-code.png "Even主题无法初始化图表") 

# 修改说明

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

# flowchart图表

## 修改过程

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
  	{{- if .Params.flowchartDiagrams.options -}}
  	  window.flowchartDiagramsOptions = {{ .Params.flowchartDiagrams.options | safeJS }};
  	{{- else if .Site.Params.flowchartDiagrams.options -}}
  	  window.flowchartDiagramsOptions = {{ .Site.Params.flowchartDiagrams.options | safeJS }};
  	{{- end -}}
      <!-- below is newly added code -->
  	let flowPageOptions = {{ .Page.Params.flowchartDiagrams.options }};
  	let flowSiteOptions = {{ .Site.Params.flowchartDiagrams.options }};
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

* 在对应的`markdown`页面头部开启`flowchart`的展示，可根据实际情况添加自定义配置，保存对应`markdown`文件后页面会自动刷新并展示对应效果[^2]。

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

## 关于自定义样式的展示



## 展示效果

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

### 图表1

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

### 图表2

### 图表3

### 图表4



# flowchart图表1

* 图表1

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

  

* 图表2

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

  

* 图表3

  ```flow
  st=>start: Start|past:>http://www.google.com[blank]
  e=>end: End:>http://www.google.com
  op1=>operation: My Operation|past
  op2=>operation: Stuff|aaa
  sub1=>subroutine: My Subroutine|invalid
  cond=>condition: Yes
  or No?|approved:>http://www.google.com
  c2=>condition: Good idea|rejected
  io=>inputoutput: catch something...|aaa
  
  st->op1(right)->cond
  cond(yes, right)->c2
  cond(no)->sub1(left)->op1
  c2(yes)->io->e
  c2(no)->op2->e
  ```

  

# sequence图表1

* 图表1

  ```sequence
  Title: Here is a title
  A->B: Normal line
  B-->C: Dashed line
  C->>D: Open arrow
  D-->>A: Dashed open arrow
  ```

* 图表2

  ```sequence
  # Example of a comment.
  Note left of A: Note to the\n left of A
  Note right of A: Note to the\n right of A
  Note over A: Note over A
  Note over A,B: Note over both A and B
  ```

* 图表3

  ```sequence
  participant C
  participant B
  participant A
  Note right of A: By listing the participants\n you can change their order
  ```

* 图表4

  ```sequence
  Andrew->China: Says Hello
  Note right of China: China thinks\nabout it
  China-->Andrew: How are you?
  Andrew->>China: I am good thanks!
  ```

* 图表5

  ```sequence
  participant System
  participant App
  System->>App: Do you hear me
  App-->>Module: Alive?
  Module-->>App: Yay!
  App->>System: Stop
  ```

# mermaid图表1

* 图表1

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

* 图表2

  ```mermaid
  graph TD
    A[Christmas] -->|Get money| B(Go shopping)
    B --> C{Let me think}
    C -->|One| D[Laptop]
    C -->|Two| E[iPhone]
    C -->|Three| F[fa:fa-car Car]
  ```

* 图表3

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

* 图表4

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

* 图表5

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

* 图表6

  ```mermaid
  gantt
      title Git Issues - days since last update
      dateFormat  X
      axisFormat %s
  
      section Issue19062
      71   : 0, 71
      section Issue19401
      36   : 0, 36
      section Issue193
      34   : 0, 34
      section Issue7441
      9    : 0, 9
      section Issue1300
      5    : 0, 5
  ```

* 图表7

  ```mermaid
  pie
  "Dogs" : 386
  "Cats" : 85.9
  "Rats" : 15
  ```

* 图表8

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

* 图表9

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

  

* 图表10

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

* 图表11

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

  

* 图表12

  ```mermaid
          timeline
          title 我的日常
          section 努力搬砖
            上午 : 早会: 收邮件/回复邮件 : 查看线上问题
            下午 : 需求评审会 : 小组周会: coding
            晚上: 加班coding
  ```

  

* 图表13

参考文章:

1. https://snowdreams1006.github.io/write/mermaid-flow-chart.html

[^1]:https://github.com/olOwOlo/hugo-theme-even/issues?q=is%3Aissue+is%3Aclosed
[^2]:https://gohugo.io/getting-started/configuration-markup/
[^3]: 此处假设我们采用`hugo server -w -D`来开启草稿模式和动态监测模式

