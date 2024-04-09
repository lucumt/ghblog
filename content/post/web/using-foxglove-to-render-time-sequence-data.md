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
                'cyan': {'fill': 'cyan'},
                'tomato': {'fill': 'tomato'},
                'darkcyan': {'fill': 'darkcyan'}
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

# 需求背景

在自动驾驶相关的项目中经常会对采集的时序数据进行可视化播放展示，常见展示方式包括`纯文本`、`图片`、`表格`、`图表`、`地图`、`3D`等展示形式，之前团队内部对于这种需求都是前后端配合自己开发相关的UI组件进行展示。但随着要展示数据的增多以及用户需求的多变，采用自行开发的方式已经力不从心，迫切的需要一款能同时支持多路数据以不同方式播放展示的工具来减轻研发压力，提升系统稳定性与可靠性。

在切换为新的可视化工具之前，内部确定了如下几个指标：

1. 能同时支持多路数据播放，可方便的添加与配置
2. 支持`纯文本`、`表格`、`图表`、`地图`、`3D`、`ROS`、`图片`、`视频`等形式播放
3. 相关实现方案不是很小众，在有使用问题时能有相关途径寻找解决方案
4. 软件License许可能允许商用，且软件代码开源以便能进行二次开发

# 软件概览

经过多方对比以及参考业界其它公司相关的方案后，最终决定采用基于`Foxglove`作为对应的可视化工具替代实现，其具有如下特性：

1. 核心是基于`React`实现的浏览器播放，`server`端只需根据规范返回对应格式的数据即可，不限制`server`端的编程语言和实现方式，前后端通信主要基于`WebSocket`实现
2. 预先定义了几十种数据格式，基于这些数据格式和实际业务需求可组合成多种不同的数据播放源，每个数据播放源都对应为一个`panel`
3. 支持多种类型的播放形式，除了前述的几种类型，其还支持`滑块`、`服务`，`日志`、`状态图`、`主题图`等十几种样式，且可根据实际需求自行开发第三方的插件并很容易的整合到系统中，在`Foxglove`实际播放时每种播放类型都对应为一个`panel`
4. 支持多通道播放，可对同一类型的`panel`可基于多个不同的数据源创建多个不同的`panel`来同时播放，也可同时配置多个不同类型的`panel`进行同时播放
5. 支持界面自定义布局与调整，可根据实际需求灵活调整不同的`panel`的展示位置以及宽高等信息，当关闭某个`panel`时整个播放界面会自动调整已达到最佳显示效果
6. 在创建`panel`并关联`server`端相关的`topic`或关闭配置好的`panel`后都会给`server`端发送对应的通知，方便进行`server`端开启或关闭数据推送服务，减少不必要的压力。同时，重复配置的`panel`只会建立一个连接，减少页面端的负载
7. 软件许可基于[**Mozilla Public License 2.0**](https://www.mozilla.org/en-US/MPL/2.0/)，可用于商业化的产品中[^1]
8. 支持在线版本和本地版本，在线版本需要付费，本地版本可通过`Docker`安装

`Foxglove`的使用流程如下图所示：

1. 首先要创建对应的`topic`,每个`topic`都代表某种类型的展示数据，除了纯文本和表格之外，对于地图、3D、图片的展示方式需要根据官方文档设置对应的`schema`
2. `topic`创建完毕后要进行注册，只有注册完毕的`topic`才能被对应的`panel`使用
3. 创建`panel`，`panel`是数据展示的基本单位，其中纯文本和表格类型可兼容所有的`topic`，而对于地图、3D、图片类型则需要与特定`schema`的`topic`关联
4. 创建完`panel`后，需要将其关联到对应的`topic`，并根据实际情况进行针对性的设置，如3D场景下设置显示的物体，地图场景下设置使用的地图源
5. 启动对应的server端，持续不停的给相关`topic`写入数据
6. 若一切正常，在`Foxglvoe`前端界面上对应的`panel`中会实时[^2]的展示server端发送来的数据
7. 若不需要某个`panel`，直接关闭即可，此时`Foxglove`会自动关闭相关的`WebSocket`连接

```flow
start=>start: 开始
end=>end: 结束
create_topic=>operation: 创建topic | pink
register_topic=>operation: 注册topic | pink
create_panel=>operation: 创建panel | cyan
config_panel=>operation: 配置panel | cyan
send_data=>operation: server端发送数据 | peru
display_data=>operation: foxglove展示 | peru
close_panel=>operation: 关闭panel | cyan
close_connection=>operation: 断开连接 | cyan
start->create_topic(right)->register_topic->create_panel(right)->config_panel->send_data(right)->display_data
display_data->close_panel->close_connection(right)->end
```

在个人项目使用中`Server`端的数据来源是基于`Kafka`实现的，相关的流程如下

```flow
start=>start: 开始
set_server=>inputoutput: 配置服务端 | pink
create_panel=>operation: 创建面板 | pink
select_chassis=>inputoutput: 选择底盘号 | tomato
fetch_data=>subroutine: 拉取kafka数据 | cyan
send_data=>subroutine: 发送kafka数据 | cyan
filter_data=>subroutine: 过滤kafka数据 | darkcyan
render_data=>operation: 展示数据 | peru
close_browser=>operation: 关闭浏览器 | pink

start->set_server(right)->create_panel->select_chassis
select_chassis->fetch_data->filter_data
filter_data(right)->send_data(right)->render_data
render_data->select_chassis
```

# 前后端配置

本章节以UI端本地私有化安装为例，说明如何配置`Foxglove`前后端通信的环境。

## 服务端安装

1. 根据`Foxglove`官方网站的[**说明文档**](https://docs.foxglove.dev/docs/visualization/message-schemas/introduction/)以及相关的[**Java Demo**](https://github.com/foxglove-custom/foxglove-websocket-java)编写对应的`server`端项目，暴露`WebSocket`端口，此处假设其端口为8765
2. 启动`server`端程序，则对应的`WebSocket`访问地址为`http://127.0.0.1:8765`

## 页面端安装

1. 在`Foxglove`对应的[**GitHub地址**](https://github.com/foxglove/studio)上有其UI端的安装说明，主要采用`Docker`安装，相关的指令如下[^3]

   ```bash
   docker run --rm -p "8080:8080" ghcr.io/foxglove/studio:latest
   ```

2. 上述指令安装完成之后，在浏览器中输入`http://127.0.0.1:8080`会打开类似如下界面，在`打开数据源`对话框中选择最下面的`打开连接`

   ![foxglove初次打开](/blog_img/web/using-foxglove-to-render-time-sequence-data/foxglove-init-open.png "foxglove初次打开")

3. 在弹出的对话框中添加前面使用的`Websocket`地址，之后点击`open`按钮关闭该对话框

   ![foxglove设置连接](/blog_img/web/using-foxglove-to-render-time-sequence-data/foxglove-set-connection.png "foxglove设置连接")

4. 若连接配置正确，在左侧会出现相关的topic列表，类似如下图所示，至此`Foxglove`前后端通信的环境初步搭建完毕

   ![foxglove正常连接](/blog_img/web/using-foxglove-to-render-time-sequence-data/foxglove-connection-config-result.png "foxglove正常连接")

# 服务器端编写

由于`Foxglove`官方已经提供了相应的`Docker`镜像可直接运行，除非需要自定义开发插件，通常不涉及到对UI端的操作，我们使用`Foxglove`时更多的工作还是集中在`server`端。

本文相关的代码参见[**foxglove-websocket-java**](https://github.com/foxglove-custom/foxglove-websocket-java)。

## topic创建

1. `Foxglove`前后端通信主要基于`WebSocket`实现，本文采用`Netty`提供的`WebSocket`工具类来简化使用，对应的`Maven`版本为

   ```xml
   <dependency>
       <groupId>org.yeauty</groupId>
       <artifactId>netty-websocket-spring-boot-starter</artifactId>
       <version>0.12.0</version>
   </dependency>
   ```

2. 添加一个如下的类用于暴露`WebSocket`端口，此时在程序启动后前后端已具备初步的通信能力

   ```java
   @Slf4j
   @ServerEndpoint(port = "8767")
   public class FoxgloveServer {
   
       public static final String EMPTY_CHASSIS_CODE = "NaN";
   
       @BeforeHandshake
       public void handshake(Session session) {
           log.info("----------session信息" + session.toString());
           session.setSubprotocols("foxglove.websocket.v1");
       }
   
       @OnOpen
       public void onOpen(Session session) {
           log.info("这是一次新的链接 ---》 new connection" + session.hashCode());
       }
   
       @OnClose
       public void onClose(Session session) throws IOException {
           // 从对象集合中删除该连接对象
           log.info("-------one connection closed");
           session.close();
       }
   
       @OnError
       public void onError(Session session, Throwable throwable) {
           throwable.printStackTrace();
       }
   
       @OnMessage
       public void onMessage(Session session, String message) {
           JSONObject msg = JSON.parseObject(message);
           String op = msg.getString("op");
           log.info("-------------on open msg:\t" + message);
       }
   
       @OnBinary
       public void onBinary(Session session, byte[] bytes) {
           // 这里接收到用户指令
           String data = new String(Arrays.copyOfRange(bytes, 5, bytes.length));
           JSONObject message = JSON.parseObject(data);
           log.info("--------binary message:\t" + message);
       }
   
   }
   ```

3. 测试

4. 测试

5. 测试

## 数据发送

# 相关功能说明

[^1]: 基于`Mozilla Public License 2.0`协议，若对其源码进行二次开发，也需要遵守同样的协议
[^2]: 此处的实时依赖于server端发送数据的频率
[^3]: 在2024年3月份原作者更新了`REAME.md`，在此次更新中将`Docker`安装的指令移除了，不过我们可通过[历史记录](https://github.com/foxglove/studio/pull/7534/files)去查找相关指令