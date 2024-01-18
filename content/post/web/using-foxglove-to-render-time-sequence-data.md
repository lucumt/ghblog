---
title: "利用Foxglove以多种方式展示时间连续的数据"
date: 2023-11-13T14:08:39+08:00
lastmod: 2023-11-13T14:08:39+08:00
draft: true
keywords: ["foxglove","时序数据","自动驾驶"]
description: "简要介绍如何利用Foxglove以不同的方式来播放展现连续的时间数据"
tags: ["web","foxglove"]
categories: ["Web编程"]
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
  enable: true
  options: "{
              'x': 10,
              'y': 10,
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
                'pink': {'fill': 'pink'},
                'peru': {'fill': 'peru'},
                'cyan': {'fill': 'cyan'}
                 }
              }"

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

简要介绍在自动驾驶相关项目中利用[**Foxglove**](https://foxglove.dev)可视化工具以多种方式播放采集的[**时序数据**](https://xie.infoq.cn/article/9783fa40d25a0a5ddbfd62d29)的使用经验分享。

<!--more-->

# 背景

在自动驾驶相关的项目中经常会对采集的时序数据进行可视化播放展示，常见展示方式包括`纯文本`、`图片`、`表格`、`图表`、`地图`、`3D`等展示形式，之前团队内部对于这种需求都是前后端配合自己开发相关的UI组件进行展示。但随着要展示数据的增多以及用户需求的多变，采用自行开发的方式已经力不从心，迫切的需要一款能同时支持多路数据以不同方式播放展示的工具来减轻研发压力，提升系统稳定性与可靠性。

在切换为新的可视化工具之前，内部确定了如下几个指标：

1. 能同时支持多路数据播放
2. 支持纯文本、表格、图表、地图、3D、ROS、图片、视频等形式播放
3. 相关实现方案不是很小众，在有使用问题时能有相关途径寻找解决方案
4. 软件License许可能允许商用，且软件代码开源以便能进行二次开发

经过多方对比以及参考业界其它公司相关的方案后，最终决定采用基于`Foxglove`作为对应的可视化工具替代实现，其具有如下特性：

1. 支持
2. 支持
3. 支持
4. 支持

`Foxglove`的使用流程如下图所示：

1. 首先要创建对应的topic,每个topic都代表某种类型的展示数据，除了纯文本这种数据之外，对于地图、3D、图片的展示方式需要根据官方文档设置对应的schema
2. topic创建完毕后要进行注册，只有注册完毕的topic才能被对应的panel使用
3. 创建panel

```flow
start=>start: 开始
end=>end: 结束
create_topic=>operation: 创建topic | pink
register_topic=>operation: 注册topic | pink
create_panel=>operation: 创建panel | cyan
send_data=>operation: 后端发送数据 | peru
display_data=>operation: foxglove展示 | peru
close_panel=>operation: 关闭panel | cyan
close_connection=>operation: 断开连接 | cyan
start->create_topic(right)->register_topic(right)->create_panel->send_data(right)->display_data
display_data(right)->close_panel->close_connection(right)->end
```



# topic注册

# 消息发送

# 相关功能说明