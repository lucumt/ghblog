---
title: "搭建基于Foxglove的时序数据播放环境"
date: 2023-11-13T14:08:39+08:00
lastmod: 2023-11-13T14:08:39+08:00
draft: false
keywords: ["foxglove","时序数据","自动驾驶"]
description: "简要介绍如何从头开始搭建基于Foxglove的时序数据播放环境"
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

简要介绍如何配置[**Foxglove**](https://foxglove.dev)可视化工具的前后端环境以便实现在项目中以多种方式播放采集的[**时序数据**](https://xie.infoq.cn/article/9783fa40d25a0a5ddbfd62d29)。

<!--more-->

## 需求背景

在自动驾驶相关的项目中经常会对采集的时序数据进行可视化播放展示，常见展示方式包括`纯文本`、`图片`、`表格`、`图表`、`地图`、`3D`等展示形式，之前团队内部对于这种需求都是前后端配合自己开发相关的UI组件进行展示。但随着要展示数据的增多以及用户需求的多变，采用自行开发的方式已经力不从心，迫切的需要一款能同时支持多路数据以不同方式播放展示的工具来减轻研发压力，提升系统稳定性与可靠性。

在切换为新的可视化工具之前，内部确定了如下几个指标：

1. 能同时支持多路数据播放，可方便的添加与配置
2. 支持`纯文本`、`表格`、`图表`、`地图`、`3D`、`ROS`、`图片`、`视频`等形式播放
3. 相关实现方案不是很小众，在有使用问题时能有相关途径寻找解决方案
4. 软件License许可能允许商用，且软件代码开源以便能进行二次开发

## 软件概览

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

## 前后端配置

本章节以UI端本地私有化安装为例，说明如何配置`Foxglove`前后端通信的环境，`server`端采用`Java`实现。

### 服务端安装

1. 根据`Foxglove`官方网站的[**说明文档**](https://docs.foxglove.dev/docs/visualization/message-schemas/introduction/)以及相关的[**Java Demo**](https://github.com/foxglove-custom/foxglove-websocket-java)编写对应的`server`端项目，暴露`WebSocket`端口，此处假设其端口为8765
2. 启动`server`端程序，则对应的`WebSocket`访问地址为`http://127.0.0.1:8765`

### 页面端安装

1. 在`Foxglove`对应的[**GitHub地址**](https://github.com/foxglove/studio)上有其UI端的安装说明，主要采用`Docker`安装，相关的指令如下[^3]

   ```bash
   # 官方原始的说明
   docker run --rm -p "8080:8080" ghcr.io/foxglove/studio:latest
   
   # 个人新保存的镜像
   docker run --rm -p "8080:8080"  lucumt/foxglove_studio:1.74.1
   ```

2. 上述指令安装完成之后，在浏览器中输入`http://127.0.0.1:8080`会打开类似如下界面，在`打开数据源`对话框中选择最下面的`打开连接`

   ![foxglove初次打开](/blog_img/web/using-foxglove-to-render-time-sequence-data/foxglove-init-open.png "foxglove初次打开")

3. 在弹出的对话框中添加前面使用的`Websocket`地址，之后点击`open`按钮关闭该对话框

   ![foxglove设置连接](/blog_img/web/using-foxglove-to-render-time-sequence-data/foxglove-set-connection.png "foxglove设置连接")

4. 若连接配置正确，在左侧会出现相关的topic列表，类似如下图所示，至此`Foxglove`前后端通信的环境初步搭建完毕

   ![foxglove正常连接](/blog_img/web/using-foxglove-to-render-time-sequence-data/foxglove-connection-config-result.png "foxglove正常连接")

## 服务器端编写

由于`Foxglove`官方已经提供了相应的`Docker`镜像可直接运行，除非需要自定义开发插件，通常不涉及到对UI端的操作，我们使用`Foxglove`时更多的工作还是集中在`server`端。

本文相关的完整代码参见[**foxglove-websocket-java**](https://github.com/foxglove-custom/foxglove-websocket-java)。

### topic创建

1. `Foxglove`前后端通信主要基于`WebSocket`实现，本文采用`Netty`提供的`WebSocket`工具类来简化使用，对应的`Maven`版本为

   ```xml
   <dependency>
       <groupId>org.yeauty</groupId>
       <artifactId>netty-websocket-spring-boot-starter</artifactId>
       <version>0.12.0</version>
   </dependency>
   ```

2. 添加一个名为`FoxgloveServer`的类用于暴露`WebSocket`端口，此时在程序启动后前后端已具备初步的通信能力

   ```java
   @Slf4j
   @ServerEndpoint(port = "8765")
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

3. 建立一个名为`ChannelInfo`的`topic`类，代码如下所示，需要注意的是类名必须为`ChannelInfo`，否则最后生成的`JSON`格式错误会导致`topic`无法注册[^4]

   ```java
   @Data
   public class ChannelInfo {
   
       /**
        * topic id，在注册时指定其值
        */
       private Integer id;
   
       /**
        * topic名称
        */
       private String topic;
   
       /**
        * topic编码，如json,protobuf等
        */
       private String encoding;
   
       /**
        * topic描述
        */
       private String schemaName;
   
       /**
        * topic对应的数据结构，可以是自定义的，也可以是foxglove官方定义好的
        */
       private String schema;
   
       /**
        * 对用结构的表示形式，如json时为jsonschema
        */
       private String schemaEncoding;
   }
   ```

4. 添加一个名为`ChannelUtil`的类，用于创建对应的`topic`

   ```java
   public class ChannelUtil {
   
       public static List<ChannelInfo> createChannels() {
           String schema;
   
           ChannelInfo channelChassis = new ChannelInfo();
           channelChassis.setId(0);
           channelChassis.setTopic("/drive/chassis_code");
           channelChassis.setEncoding("json");
           channelChassis.setSchemaName("底盘编码(用于切换播放源)");
           channelChassis.setSchema("{\"type\": \"object\", \"properties\": {\"chassis_code\": {\"type\": \"string\"}}}");
           channelChassis.setSchemaEncoding("jsonschema");
   
           ChannelInfo channelControl = createControlData(1);
   
           ChannelInfo channel3D = new ChannelInfo();
           channel3D.setId(2);
           channel3D.setTopic("/drive/3D");
           channel3D.setEncoding("json");
           channel3D.setSchemaName("foxglove.SceneUpdate");
           schema = DataUtil.loadJsonSchema("SceneUpdate.json");
           channel3D.setSchema(schema);
           channel3D.setSchemaEncoding("jsonschema");
   
           ChannelInfo channelGPS = new ChannelInfo();
           channelGPS.setId(3);
           channelGPS.setTopic("/drive/map");
           channelGPS.setEncoding("json");
           channelGPS.setSchemaName("foxglove.LocationFix");
           schema = DataUtil.loadJsonSchema("LocationFix.json");
           channelGPS.setSchema(schema);
           channelGPS.setSchemaEncoding("jsonschema");
   
   
           List<ChannelInfo> channelList = new ArrayList<>();
           channelList.add(channelChassis);
           channelList.add(channelControl);
           channelList.add(channel3D);
           channelList.add(channelGPS);
           return channelList;
       }
   
       private static ChannelInfo createControlData(int index) {
           ChannelInfo channelControl = new ChannelInfo();
           channelControl.setId(index);
           channelControl.setTopic("/drive/control_data");
           channelControl.setEncoding("json");
           channelControl.setSchemaName("控制数据信号");
           StringBuffer sb = new StringBuffer();
           sb.append("{\"type\": \"object\", \"properties\":{");
           sb.append("\"底盘号\": {\"type\": \"string\"},");
           sb.append("\"终端id\": {\"type\": \"string\"},");
           sb.append("\"时间\": {\"type\": \"string\"},");
           sb.append("\"纬度\": {\"type\": \"string\"},");
           sb.append("\"经度\": {\"type\": \"string\"},");
           sb.append("\"海拔\": {\"type\": \"string\"},");
           sb.append("\"变速箱输出轴转速\": {\"type\": \"string\"},");
           sb.append("\"制动系统准备可以释放\": {\"type\": \"string\"},");
           sb.append("\"角速度(rad/s)\": {\"type\": \"string\"},");
           sb.append("\"AX(m/s^2)\": {\"type\": \"string\"},");
           sb.append("\"AY(m/s^2)\": {\"type\": \"string\"},");
           sb.append("\"总驱动力\": {\"type\": \"string\"}");
           sb.append("}}");
           channelControl.setSchema(sb.toString());
           channelControl.setSchemaEncoding("jsonschema");
           return channelControl;
       }
   
   }
   ```

5. 对前述的`FoxgloveServer`类中的`onOpen`方法添加类似如下代码，用于真正的注册相关的`topic`，可注册的`topic`数目可根据实际需求添加任意多个

   ```java
   @OnOpen
   public void onOpen(Session session) {
       log.info("这是一次新的链接 ---》 new connection" + session.hashCode());
   
       this.session = session;
   
       ServerInfo serverInfo = new ServerInfo();
       serverInfo.setOp("serverInfo");
       serverInfo.setName("foxglove data render");
       serverInfo.setCapabilities(Arrays.asList("clientPublish", "services"));
       serverInfo.setSupportedEncodings(Arrays.asList("json"));
   
       String severInfoString = JSON.toJSONString(serverInfo);
   
       session.sendText(severInfoString);
       Advertise advertise = new Advertise();
       advertise.setOp("advertise");
       advertise.setChannels(ChannelUtil.createChannels());
   
       session.sendText(JSON.toJSONString(advertise));
   }
   ```

6. 若连接配置正确，在UI界面的左侧会出现相关的`topic`列表，如前述步骤所示，至此`topic`的创建与注册完成

### 数据发送

本章节以发送`纯文本`类型的消息为例说明如何在`server`端编写代码实现数据发送

1. `topic`注册完毕后，不能直接使用，需要创建`panel`并选中对应的`topic`，此时UI端会给`server`端通过`WebSocket`协议发送对应的事件消息，需修改`FoxgloveServer`中的`onMessage`方法

   ```java
   @OnMessage
   public void onMessage(Session session, String message) {
       JSONObject msg = JSON.parseObject(message);
       String op = msg.getString("op");
       log.info("-------------on open msg:\t" + message);
       switch (op) {
           case "subscribe":
               // 创建连接时会执行此处逻辑
               this.createThread(msg);
               break;
           case "unsubscribe":
               break;
       }
   }
   
   private void createThread(JSONObject msg) {
       List<Subscription> subscribeList = msg.getObject("subscriptions", new TypeReference<List<Subscription>>() {
       });
       log.info("============开始创建基于channel的数据发送线程==============" + subscribeList);
       for (Subscription sub : subscribeList) {
           Integer channelId = sub.getChannelId();
           SendDataThread thread = getKafkaSendThread(sub.getId(), channelId, session);
           String threadName = "thread-" + getTopicName(channelId) + "-" + RandomStringUtils.randomAlphabetic(6).toLowerCase();
           thread.setName(threadName);
           thread.setChassisCode(chassisCode == null ? EMPTY_CHASSIS_CODE : chassisCode);
           thread.start();
           threadMap.put(channelId, thread);
       }
   }
   ```

   其中`SendDataThread`类是一个继承自`Thread`类，用于封装`Kafka`的数据发送流程，具体的发送逻辑需要再次继承`SendDataThread`类来实现。

2. 在前述`topic`列表中`纯文本`对应的名称为`/drive/chassis_code`，对应的数据结构如下

   ```json
   {
       "type": "object",
       "properties": {
           "chassis_code": {
               "type": "string"
           }
       }
   }
   ```

   则可继承`SendDataThread`编写相应的实现类

   ```java
   @Slf4j
   public class SendChassisThread extends SendDataThread {
       
       private DataConfig dataConfig;
   
       public SendChassisThread(int index, Session session) {
           super(index, session);
           this.dataConfig = AppCtxUtil.getBean(DataConfig.class);
           this.frequency = dataConfig.getChassis().getFrequency();
       }
   
       @Override
       public void run() {
           while (running) {
               try {
                   ChassisInfo chassis = new ChassisInfo();
                   chassis.setTimestamp(DateUtil.createTimestamp());
                   chassis.setChassisCode(RandomStringUtils.randomAlphanumeric(6).toUpperCase());
                   JSONObject jsonObject = (JSONObject) JSONObject.toJSON(chassis);
                   byte[] bytes = jsonObject.toJSONString().getBytes();
                   this.session.sendBinary(bytes);
                   log.info("---------------chassis info:\t" + chassis);
                   sleep(frequency);
               } catch (InterruptedException e) {
                   throw new RuntimeException(e);
               }
           }
       }
   }
   ```

   对应的`ChassisInfo`类代码如下

   ```java
   @Data
   public class ChassisInfo {
   
       @JsonProperty("底盘号")
       private String chassisCode;
   
       @JsonProperty("时间戳")
       private Timestamp timestamp;
   }
   ```

3. 启动`server`端程序，在`Foxglove`UI端按照下图所示点击创建按钮，在出现的`panel`列表中选择**原始消息**(英文环境下为**Raw Message**)，用于展示纯文本消息

   ![foxglove创建面板](/blog_img/web/using-foxglove-to-render-time-sequence-data/foxglove-create-panel.png "foxglove创建面板")

4. 在出现的纯文本`panel`的顶部选择对应的`topic`，此处为`/drive/chassis_code`，执行到这一步后理论上就能正常播放数据，但在浏览器中查看结果时，会呈现出类似下图所示的效果，文本消息并没有正常展示。

   ![foxglove不能正常展示消息](/blog_img/web/using-foxglove-to-render-time-sequence-data/foxglove-can-not-display-text-message.png "foxglove不能正常展示消息")

5. 从上面报错的初步信息可知是UI端在解析二进制文件时出错，与官方Demo的`WebSocket`数据包进行对比，折腾一段时间后其原因是`Foxglove`有自己特定的编码算法，不能直接返回生成的原始消息，需要对其基于`byte`进行编码，之后UI端自行解码。相关的编码代码如下

   ```java
   package com.visualization.foxglove.util;
   
   import com.alibaba.fastjson.JSON;
   import com.fasterxml.jackson.databind.ObjectMapper;
   import org.apache.commons.io.IOUtils;
   
   import java.io.IOException;
   import java.io.InputStream;
   import java.util.Map;
   
   public class DataUtil {
   
       public static byte[] getFormattedBytes(byte[] data, int channel) {
           return getFormattedBytes(data, 0, channel);
       }
   
       public static byte[] getFormattedBytes(byte[] data, long ns, int channel) {
           byte constantInfo = 1;
           byte[] constantInfoByte = new byte[]{constantInfo};
           byte[] dataType = getIntBytes(channel);
           byte[] nsTime = getLongBytes(ns);
           byte[] pack1 = byteConcat(constantInfoByte, dataType, nsTime);
           byte[] pack2 = byteConcat(pack1, data);
           return pack2;
       }
   
       public static byte[] loadGlbData(String glbFile) {
           try {
               ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
               InputStream stream = classLoader.getResourceAsStream("glb/" + glbFile);
               byte[] bytes = IOUtils.toByteArray(stream);
               return bytes;
           } catch (IOException e) {
               e.printStackTrace();
           }
           return null;
       }
   
       public static String loadJsonSchema(String schemaFile) {
           ObjectMapper objectMapper = new ObjectMapper();
           Map map = null;
           try {
               ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
               InputStream stream = classLoader.getResourceAsStream("schema/" + schemaFile);
               map = objectMapper.readValue(stream, Map.class);
           } catch (IOException e) {
               e.printStackTrace();
           }
           return JSON.toJSONString(map);
       }
   
       public static final long fx = 0xffL;
   
       /**
        * int 转 byte[]
        * 小端
        *
        * @param data
        * @return
        */
       public static byte[] getIntBytes(int data) {
           int length = 4;
           byte[] bytes = new byte[length];
           for (int i = 0; i < length; i++) {
               bytes[i] = (byte) ((data >> (i * 8)) & fx);
           }
           return bytes;
       }
   
       /**
        * long 转 byte[]
        * 小端
        *
        * @param data
        * @return
        */
       public static byte[] getLongBytes(long data) {
           int length = 8;
           byte[] bytes = new byte[length];
   
           for (int i = 0; i < length; i++) {
               bytes[i] = (byte) ((data >> (i * 8)) & fx);
           }
           return bytes;
       }
   
       public static byte[] byteConcat(byte[] bt1, byte[] bt2) {
           byte[] bt4 = new byte[bt1.length + bt2.length];
           int len = 0;
           System.arraycopy(bt1, 0, bt4, 0, bt1.length);
           len += bt1.length;
           System.arraycopy(bt2, 0, bt4, len, bt2.length);
           return bt4;
       }
   
       public static byte[] byteConcat(byte[] bt1, byte[] bt2, byte[] bt3) {
           byte[] bt4 = new byte[bt1.length + bt2.length + bt3.length];
           int len = 0;
           System.arraycopy(bt1, 0, bt4, 0, bt1.length);
           len += bt1.length;
           System.arraycopy(bt2, 0, bt4, len, bt2.length);
           len += bt2.length;
           System.arraycopy(bt3, 0, bt4, len, bt3.length);
           return bt4;
       }
   
   }
   ```

6. 将数据发送的代码修改如下，并重新启动`server`端服务

   ```java
   @Override
   public void run() {
       while (running) {
           try {
               ChassisInfo chassis = new ChassisInfo();
               chassis.setTimestamp(DateUtil.createTimestamp());
               chassis.setChassisCode(RandomStringUtils.randomAlphanumeric(6).toUpperCase());
               JSONObject jsonObject = (JSONObject) JSONObject.toJSON(chassis);
               // 需要调用对应的编码函数
               byte[] bytes = DataUtil.getFormattedBytes(jsonObject.toJSONString().getBytes(), index);
               this.session.sendBinary(bytes);
               log.info("---------------chassis info:\t" + chassis);
               sleep(frequency);
           } catch (InterruptedException e) {
               throw new RuntimeException(e);
           }
       }
   }
   ```

7. 在UI端查看，可发现此时能正常播放消息，至此`Foxglove`前后端交互的配置基本搭建完毕，后续就是根据不同的面板类型进行针对性的编码。

   ![foxglove正常展示消息](/blog_img/web/using-foxglove-to-render-time-sequence-data/foxglove-display-text-message-success.png "foxglove正常展示消息")

[^1]: 基于`Mozilla Public License 2.0`协议，若对其源码进行二次开发，也需要遵守同样的协议
[^2]: 此处的实时依赖于server端发送数据的频率
[^3]: 在2024年3月份原作者更新了`REAME.md`，在此次更新中将`Docker`安装的指令移除了，不过我们可通过[历史记录](https://github.com/foxglove/studio/pull/7534/files)去查找相关指令
[^4]: 感觉此处`Foxglove`的命名有些混乱，通道被命名为`Topic`，然而在`JSON`格式文件中却要求用`Channel`来替代