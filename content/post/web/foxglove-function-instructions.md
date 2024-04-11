---
title: "Foxglove基本功能使用说明"
date: 2023-11-15T14:08:39+08:00
lastmod: 2023-11-15T14:08:39+08:00
draft: true
keywords: ["foxglove","时序数据","自动驾驶","功能使用"]
description: "Foxglove基本功能使用说明"
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

基于[**搭建基于Foxglove的时序数据播放环境**](/post/web/using-foxglove-to-render-time-sequence-data/)一文，结合实际项目的使用情况对`Foxglove`的使用功能进行简要的介绍。

<!--more-->

# 面板操作

`Foxglove`中的数据展示是基于面板来实现的，可根据实际使用需求在浏览器窗口添加多个面板，每个面板需要配置对应的topic来展示不同的数据，它们各司其职互不干扰。

![foxglove面板展示](/blog_img/web/foxglove-function-instructions/foxglove_panels_display.png "foxglove面板展示")

自己项目中用到的面板主要有如下6种：

| 面板         | 作用                                                   | 关联topic[^1]        | 备注                           |
| ------------ | ------------------------------------------------------ | -------------------- | ------------------------------ |
| **发布面板** | 与`server`端进行交互，如根据输入的底盘号动态切换数据   | `drive/chassis_code` | 该面板不做展示用               |
| **文本面板** | 以纯文字形式展示车辆数据                               | 所有topic均可        | 建议使用`drive/control_data`   |
| **图表面板** | 以图表形式展示车辆数据                                 | 所有topic均可        |                                |
| **地图面板** | 用于在地图中展示车辆的行驶轨迹                         | `drive/map`          |                                |
| **3D面板**   | 用于显示**3D**场景下的`车道线`、`障碍物`以及`车辆`本身 | `drive/3D`           |                                |
| **视频面板** | 视频展示                                               | 不需要               | 直接通过`RTSP`视频流的形式播放 |

可在`Foxglove`系统中进行面板的创建、修改、删除、调整、导入、导出等操作，实际使用时需要在`server`端根据`Foxglove`规范提前准备好相关的`topic`。

## 创建面板

### 直接创建

1. 在浏览器窗口的左上角点击创建按钮，在出现的菜单中选择对应的类型

   ![foxglove创建面板](/blog_img/web/foxglove-function-instructions/foxglove-create-panel.png "foxglove创建面板")

2. 如下图所示直接创建的面板会占用左侧空间，挤占已有区域，当已有多个面板展示时不建议采用此种方式

   ![foxglove直接创建面板](/blog_img/web/foxglove-function-instructions/foxglove-direct-create-panel-result.png "foxglove直接创建面板")

3. 面板创建完毕后，根据不同的面板类型需要做相关配置操作(见下文)

### 拆分创建

1. 选中一个已有的面板，在其右上角点击如下图所示的按钮，根据需求选择拆分类型

   ![foxglove拆分面板](/blog_img/web/foxglove-function-instructions/foxglove-split-panel.png "foxglove拆分面板")

2. 假设选择"向下拆分"，拆分后的结果如下所示，可看出会创建一个相同的面板且继承原有面板的各种配置

   ![foxglove拆分面板结果](/blog_img/web/foxglove-function-instructions/foxglove-split-panel-result.png "foxglove拆分面板结果")

3. 如下图所示，在拆分后的新面板中继续点击右上角的按钮，在出现的菜单中选择`更改面板`，之后才出现的面板类型菜单中根据实际需要选择合适的类型

   ![foxglove拆分面板修改](/blog_img/web/foxglove-function-instructions/foxglove-split-panel-change.png "foxglove拆分面板修改")

4. 选择完毕后，需根据实际情况对面板做出相应配置才能正常显示(具体请参见下文的面板配置说明)

   ![foxglove拆分面板修改结果](/blog_img/web/foxglove-function-instructions/foxglove-split-panel-change-result.png "foxglove拆分面板修改结果")

## 修改面板

对于已有的面板可修改其配置，如展示颜色、消息来源等，具体步骤如下：

1. 测试
2. 测试
3. 测试
4. 测试
5. 测试
6. 测试

## 删除面板

## 打开工具栏

# 面板配置

## 文本

## 图表

## 地图

## 3D

## 图片/视频

# 布局操作

## 布局调整

## 导入/导出

## 全屏展示

# 动态交互

[^1]: 若需要关联特定`topic`则表示该类面板对返回的数据格式有特殊要求