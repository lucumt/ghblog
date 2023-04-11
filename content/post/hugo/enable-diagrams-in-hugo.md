---
title: "在Hugo中开启图表支持"
date: 2023-03-27T09:46:50+08:00
lastmod: 2023-03-27T09:46:50+08:00
draft: false
keywords: ["flowchart","sequence","mermaid","hugo","markdown"]
description: ""
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



# flowchart图表

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

  

# sequence图表

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

# mermaid图表

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
      Parallel 2   :         des4, after des1, 1d
      Parallel 3   :         des5, after des3, 1d
      Parallel 4   :         des6, after des4, 1d
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
          title My day
          section Go to work
            1930 : first step : second step
                 : third step
            1940 : fourth step : fifth step
  ```

  

* 图表13

参考文章:

1. https://snowdreams1006.github.io/write/mermaid-flow-chart.html