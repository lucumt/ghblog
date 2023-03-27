---
title: "在Hugo中开启图表和流程图支持"
date: 2023-03-27T09:46:50+08:00
lastmod: 2023-03-27T09:46:50+08:00
draft: true
keywords: []
description: ""
tags: ["hugo","Go"]
categories: []
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
                'request': {'fill': 'pink'},
                'approved': {'fill': 'peru'}
              }
            }"

sequenceDiagrams: 
  enable: false
  options: ""

---

<!--more-->

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
  op2=>operation: Stuff|current
  sub1=>subroutine: My Subroutine|invalid
  cond=>condition: Yes
  or No?|approved:>http://www.google.com
  c2=>condition: Good idea|rejected
  io=>inputoutput: catch something...|request
  
  st->op1(right)->cond
  cond(yes, right)->c2
  cond(no)->sub1(left)->op1
  c2(yes)->io->e
  c2(no)->op2->e
  ```

  