---
title: "在批处理中利用&&命令将多条命令优雅的拼接在一起"
date: 2023-03-10T22:48:56+08:00
lastmod: 2023-03-10T22:48:56+08:00
draft: true
keywords: ["bat","&&","批处理"]
description: "在批处理中利用&&命令将多条命令优雅的拼接在一起，在实现功能的同时保持脚本代码的格式整洁与可维护性"
tags: ["cmd"]
categories: ["脚本操作"]
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

简要介绍在`Windows`系统中采用基于`CMD`的批处理脚本进行自动化部署时，对通过`&&`拼接的指令以更优雅的方式实现，以提高批处理脚本的可读性和可维护性。

<!--more-->

# 背景

公司有个基于`Python`开发的项目近期要交付给客户，为了便于安装和后续升级，之前都是采用[PyInstaller](https://pyinstaller.org/)安装成exe可执行文件实现傻瓜式的操作，而这个项目出于避免与客户服务器其它`Python`环境冲突以及便于后续扩展的考量，采用了[Anaconda](https://www.anaconda.com/)进行隔离式部署。

# 部署方式

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

测试

```flow
st=>start: Start
e=>end: End
cond=>condition: Option
op1=>operation: solution_1
op2=>operation: solution_2

st->cond
cond(yes)->op1->e
cond(no)->op2->es
```

# 解决方案

test **abc**

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

test **Taichung goes global**

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

