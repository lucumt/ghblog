---
title: "Foxglove基本功能使用说明"
date: 2023-11-15T14:08:39+08:00
lastmod: 2023-11-15T14:08:39+08:00
draft: false
keywords: ["foxglove","时序数据","自动驾驶","功能使用"]
description: "对Foxglove基本功能进行简单的使用说明，让其他初次接触的人可快速上手"
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

## 面板操作

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

### 创建面板

#### 直接创建

1. 在浏览器窗口的左上角点击创建按钮，在出现的菜单中选择对应的类型

   ![foxglove创建面板](/blog_img/web/foxglove-function-instructions/foxglove-create-panel.png "foxglove创建面板")

2. 如下图所示直接创建的面板会占用左侧空间，挤占已有区域，当已有多个面板展示时不建议采用此种方式

   ![foxglove直接创建面板](/blog_img/web/foxglove-function-instructions/foxglove-direct-create-panel-result.png "foxglove直接创建面板")

3. 面板创建完毕后，根据不同的面板类型需要做相关配置操作(见下文)

#### 拆分创建

1. 选中一个已有的面板，在其右上角点击如下图所示的按钮，根据需求选择拆分类型

   ![foxglove拆分面板](/blog_img/web/foxglove-function-instructions/foxglove-split-panel.png "foxglove拆分面板")

2. 假设选择"向下拆分"，拆分后的结果如下所示，可看出会创建一个相同的面板且继承原有面板的各种配置

   ![foxglove拆分面板结果](/blog_img/web/foxglove-function-instructions/foxglove-split-panel-result.png "foxglove拆分面板结果")

3. 如下图所示，在拆分后的新面板中继续点击右上角的按钮，在出现的菜单中选择`更改面板`，之后才出现的面板类型菜单中根据实际需要选择合适的类型

   ![foxglove拆分面板修改](/blog_img/web/foxglove-function-instructions/foxglove-split-panel-change.png "foxglove拆分面板修改")

4. 选择完毕后，需根据实际情况对面板做出相应配置才能正常显示(具体请参见下文的面板配置说明)

   ![foxglove拆分面板修改结果](/blog_img/web/foxglove-function-instructions/foxglove-split-panel-change-result.png "foxglove拆分面板修改结果")

### 修改面板

对于已有的面板可修改其配置，如展示颜色、消息来源等，具体步骤如下：

1. 选择一个要修改的面板，点击右上角的`设置`按钮

   ![foxglove打开面板设置](/blog_img/web/foxglove-function-instructions/foxglove-open-settings-dialog.png "foxglove打开面板设置")

2. 浏览器左侧会出现该面板对应的配置界面，如下图所示

   ![foxglove面板设置](/blog_img/web/foxglove-function-instructions/foxglove-panel-with-settings-dialog.png  "foxglove面板设置")

3. 可根据实际请求修改配置，修改完毕后立即生效，下图显示的为修改按钮颜色后的效果

   ![foxglove面板配置修改](/blog_img/web/foxglove-function-instructions/foxglove-panel-with-settings-update.png  "foxglove面板配置修改")

### 删除面板

1. 在要删除的面板右上角点击`设置`按钮，在出现的菜单中点击`删除面板`即可删除该面板

   ![foxglove删除面板](/blog_img/web/foxglove-function-instructions/foxglove_delete_panel.png  "foxglove删除面板")

2. 面板删除完毕后`Foxglove`会自动调整浏览器中的界面布局，同时`Foxglove`会根据实际情况给`server`发送对应的通知，以决定是否要停止相关的线程。

### 打开工具栏

除了通过修改面板打开工具栏这种方式外，也可点击浏览器窗口右上角的按钮来打开工具栏，之后再点击对应的面板也能出现设置界面，操作步骤如下：

1. 若展示界面没有显示工具栏，可点击页面右上角的按钮，如下图所示

   ![foxglove打开工具栏](/blog_img/web/foxglove-function-instructions/foxglove-open-toolbar.png  "foxglove打开工具栏")

2. 此时由于没有选择任何面板，工具栏区域为空

   ![foxglove工具栏为空](/blog_img/web/foxglove-function-instructions/foxglove-empty-toolbar.png  "foxglove工具栏为空")

3. 单击任意一个面板后，工具栏部分会出现对应面板的设置界面，可根据实际情况进行相应的设置操作

   ![foxglove工具栏设置](/blog_img/web/foxglove-function-instructions/foxglove-panel-toolbar.png  "foxglove工具栏设置")

## 面板配置

### 文本

文本[^2]在`Foxglove`中对应的类型为`原始消息`(英文环境下为`Raw Message`)，其配置较为简单：

1. 创建好面板后，在其上方可选择对应的`topic`，可选择所有的`topic`

   ![原始消息选择topic](/blog_img/web/foxglove-function-instructions/foxglove-raw-message-select-topic.png  "原始消息选择topic")

2. 选择好之后`Foxglove`中会立即实时展示该topic中对应的数据

   ![原始消息展示所有内容](/blog_img/web/foxglove-function-instructions/foxglove-raw-message-all-property.png  "原始消息展示所有内容")

3. 若发现上述面板中现实的信息太多，可根据实际需求只展示某个参数(属性)，需要在选择topic后精确到具体的属性，如下图所示

   ![foxglove原始消息选择单个属性](/blog_img/web/foxglove-function-instructions/foxglove-raw-message-select-single-property.png  "foxglove原始消息选择单个属性")

4. 选择完毕后呈现的效果类似如下

   ![foxglove原始消息单个属性数据展示](/blog_img/web/foxglove-function-instructions/foxglove-raw-message-single-property-data.png  "foxglove原始消息单个属性数据展示")

### 图表

表格在`Foxglove`中对应的类型为`图表`(英文环境下显示为`Plot`)，其配置过程如下：

1. 创建好图表面板之后，其展示效果类似如下，由于此时没有选择任何数据，所以面板上展示为空

   ![foxglove空图表](/blog_img/web/foxglove-function-instructions/foxglove-empty-plot.png  "foxglove空图表")

2. 可在左侧配置界面的`数据系列`部分添加一个消息地址，类似文本种添加单个消息的操作，一次只能选择一个参数(属性)

   ![foxglove图表选择要展示的数据](/blog_img/web/foxglove-function-instructions/foxglove-plot-select-property.png  "foxglove图表选择要展示的数据")

3. 选择完参数后，执行效果类似下图

   ![foxglove图表单个数据展示](/blog_img/web/foxglove-function-instructions/foxglove-plot-data-show-single.png  "foxglove图表单个数据展示")

4. 可在上图的左侧设置区域点击增加按钮来添加多个统计参数，添加后的展示效果类似如下

   ![foxglove图表多个数据展示](/blog_img/web/foxglove-function-instructions/foxglove-plot-data-show-multiple.png  "foxglove图表多个数据展示")

5. 当有多个参数展示时，图表中默认悬浮显示的参数名称可能会影响观看，可点击其左侧的按钮将其折叠

   ![foxglove图表隐藏参数](/blog_img/web/foxglove-function-instructions/foxglove-plot-hide-property-name.png  "foxglove图表隐藏参数")

6. 其它的配置包括设置线条颜色、X轴、Y轴、时间设置等也比较简单与直观，此处不再详述。

### 地图

地图数据用于默认展示车辆在地图中的移动轨迹，其只能选择特定类型的`topic`，相关配置过程如下：

1. 创建完地图面板后展示效果如下，可以看出其结果空白，这是由于`Foxglove`中的地图默认采用的是`OpenStreetMap`而其在国内访问受限制，需要设置为国内的地图源

   ![foxglove地图面板不展示](/blog_img/web/foxglove-function-instructions/foxglove_map_panel_not_render.png  "foxglove地图面板不展示")

2. 打开其配置面板，按照下图所示依次设置，先将地图层改为`Custom`，然后在自定义的地图地址中输入高德地图的地址，目前可用的为`https://webrd04.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=7&x={x}&y={y}&z={z}`，执行完这一步骤后地图面板中就能正常显示

   ![foxglove地图配置](/blog_img/web/foxglove-function-instructions/foxglove_map_panel_config.png  "foxglove地图配置")

3. 可以将`Follow topic`设置为具体的`topic`(本例中为`/drive/map`)，之后地图会随着车辆的移动轨迹动态更新变化。

### 3D

3D面板同地图面板类似，只能使用特定的`topic`(本例中为`/drive/3D`)，其配置过程如下：

1. 创建完毕3D面板后其不会展示数据，同时打开配置界面会发现其在`参考系`部分报错，如下图所示，此时可在配置界面点击对应topic的眼睛按钮，将其设置为可见

   ![foxglove3D面板展示不正常](/blog_img/web/foxglove-function-instructions/foxglove_3d_panel_not_render.png  "foxglove3D面板展示不正常")

2. 操作完毕后即可正常显示3D数据，同时报错消失，由于不同的车辆采集数据不同，可在对应展示区域中通过鼠标滚轮放大或缩小

   ![foxglove3D面板展示正常](/blog_img/web/foxglove-function-instructions/foxglove_3d_panel_working.png  "foxglove3D面板展示正常")

3. 其余的场景、视图、变换等可根据实际情况进行配置操作。

### 图片/视频

截止到本文写作时(2023年11月)，`Foxglove`的官方只支持图片播放，暂不支持视频直接播放，而个人在实际使用中发现按照特定的频率对视频进行抽帧时会有一定的耗时，导致播放时比较卡顿。

考虑播放的连贯与性能，决定采用`RTSP`协议来播放视频替代图片播放，为此个人开发了一款插件[**rtsp-player-extension**](https://github.com/foxglove-custom/rtsp-player-extension)[^3]，使用时需预先在浏览器上安装该插件，然后确保对应`RTSP`视频流有数据即可正常播放。

![foxglove视频面板](/blog_img/web/foxglove-function-instructions/foxglove_video_panel.png  "foxglove视频面板")

## 布局操作

### 布局调整

如下图所示，当有多个面板时，可将鼠标放到面板边框来调整相邻面板的宽度或高度。

![foxglove面板调整大小](/blog_img/web/foxglove-function-instructions/foxglove-panel-resize.png  "foxglove面板调整大小")

### 导入/导出

> 当要在多个浏览器(电脑)上使用`Foxglove`进行显示时，若基于前述的步骤依次创建面板和导出面板的话，会显得不方便，此时可通过导入导出功能，将一个浏览器中已经配置的面板与布局导出为`json`配置文件，之后通过配置文件在另一个浏览器中快速导入。

1. 如下图所示，在配置好面板的浏览器中点击左上角的icon按钮，依次选择`查看`->`导出布局到文件`，将当前配置好的面板和样式布局都导出到一个`json`文件中

   ![foxglove导出配置文件](/blog_img/web/foxglove-function-instructions/foxglove-export-layout-file.png  "foxglove导出配置文件")

2. 当在一个新的浏览器窗口(或电脑)上使用`Foxglove`时在配置后服务器后，其默认显示的界面如下，可以看出当前没有任何配置好的面板

   ![foxglove没有面板](/blog_img/web/foxglove-function-instructions/foxglove-no-panels.png  "foxglove没有面板")

3. 将前述导出的`json`文件拷贝到目标电脑上，然后在浏览器中点击左上角的icon按钮，依次选择`查看`->`通过文件导入布局`来导入对应的`json`文件

   ![foxglove导入配置文件](/blog_img/web/foxglove-function-instructions/foxglove-import-layout-file.png  "foxglove导入配置文件")

4. 导入成功之后显示类似如下界面，需要在发布界面(若没有则手工添加)选择对应的底盘号发布，之后才能从`Kafka`中正常获取数据并显示。

   ![foxglove导入配置文件结果](/blog_img/web/foxglove-function-instructions/foxglove-with-panels.png  "foxglove导入配置文件结果")

### 全屏展示

> `Foxglove`支持单个面板的全屏展示与退出，同时不影响其它面板的正常使用。

1. 选择要全屏展示的组件，在其左上角点击设置按钮

   ![foxglove进入全屏](/blog_img/web/foxglove-function-instructions/foxglove_enter_max_screen.png  "foxglove进入全屏")

2. 在出现菜单列表中点击`全屏`即可进入全屏模式

   ![foxglove面板全屏展示](/blog_img/web/foxglove-function-instructions/foxglove_panel_max_screen.png  "foxglove面板全屏展示")

3. 如下图所示，在全屏模式下点击右上角的退出按钮，即可退出全屏模式

   ![foxglove退出全屏](/blog_img/web/foxglove-function-instructions/foxglove_exit_max_screen.png  "foxglove退出全屏")

## 动态交互

> 动态交互是指通过在`Foxglove`UI端输入相应的参数给`server`端，`server`端根据相关参数对`WebSocket`中要返回的数据进行动态生成或动态过滤。

本章节以切换车辆底盘号显示不同车辆数据的场景为例，进行说明：

1. 仿照前述步骤中的新建面板，将面板类型选择为`发布`(也可通过面板拆分创建的方式，将新面板的类型修改为`发布`)

   ![foxglove创建发布面板](/blog_img/web/foxglove-function-instructions/foxglove_create_publish_panel.png  "foxglove创建发布面板")

2. 此时左侧的工具栏会报错，在`Topic`部分选择一个合适的`topic`本例中为`drive/chassis_code`后错误会消失

   ![foxglove发布面板选择主题](/blog_img/web/foxglove-function-instructions/foxglove_publish_panel_select_topic.png  "foxglove发布面板选择主题")

3. 在面板输入区域填入类似如下内容，其中`chassis_code`对应的值为车辆底盘号，之后点击`Publish`[^4]即可更换车辆底盘号

   ```json
   {
     "chassis_code": "ND000048"
   }
   ```

   ![foxglove发布面板输入底盘号](/blog_img/web/foxglove-function-instructions/foxglove_publish_panel_input_chassis.png  "foxglove发布面板输入底盘号")

4. 发布成功之后，各个面板中的数据都会切换为对应底盘号的数据，其呈现的结果都会变化，也可直接在原始消息中查看底盘号是否发生变化。

   ![foxglove检查底盘号变更](/blog_img/web/foxglove-function-instructions/foxglove_check_chassis_publish_result.png  "foxglove检查底盘号变更")

5. 数据正常切换后，可将发布面板删除，避免干扰正常使用。

[^1]: 若需要关联特定`topic`则表示该类面板对返回的数据格式有特殊要求
[^2]: 此处主要指的是纯文字文本
[^3]: 具体请参见本人的另一篇博文[Foxglove自定义插件开发说明](/post/web/foxglove-custom-plugin-develop/)
[^4]: 此时会给`server`端发送消息，具体参见[foxglove-websocket-java](https://github.com/foxglove-custom/foxglove-websocket-java/blob/789b9dcd7d5cea16655d128d097ecc076618f767/src/main/java/com/visualization/foxglove/websocket/FoxgloveServer.java#L96)