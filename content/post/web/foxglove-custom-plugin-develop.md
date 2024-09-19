---
title: "Foxglove自定义插件开发说明"
date: 2023-11-22T14:08:39+08:00
lastmod: 2023-11-22T14:08:39+08:00
draft: false
keywords: ["foxglove","插件开发"]
description: "简要介绍如何给Foxglove进行插件开发，以满足不同的使用需求"
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
  enable: true
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

基于[**create-foxglove-extension**](https://github.com/foxglove/create-foxglove-extension)一文简要说明自己如何基于[**RTSP协议**](https://en.wikipedia.org/wiki/Real-Time_Streaming_Protocol)给`Foxglove`开发一款插件并应用于实际项目的流程。

相关源码参见[**rtsp-player-extension**](https://github.com/foxglove-custom/rtsp-player-extension)。

<!--more-->

## 问题背景

`Foxglove`官方虽然支持十几种类型的数据播放格式，但却没有对视频流播放的支持(截至到本文写作时)[^1]，而自动驾驶等相关的汽车研发领域对于视频文件的播放却是一个刚需，且通常需要播放多路视频。

由于`Foxglove`官方支持图片、压缩图片、压缩帧这几种格式的播放，故一开始自己想的是将视频按照特定的频率抽取成多帧图片，在UI端按照时间顺序连续播放形成类似动画的效果，以间接实现视频播放的目的。

在个人实际项目中，视频的来源是`RTSP`，故而最初的处理逻辑如下，需要自己实现的核心逻辑为将视频流抽帧为图片

```flow
start=>start: 开始
end=>end: 结束
connect_rtsp=>operation: 连接RTSP视频流 | pink
extract_images=>operation: 抽帧为图片 | peru
convert_images=>operation: 转化为Foxglove格式  | pink
send_data=>operation: WebSocket发送 | pink
display_data=>operation: UI端展示 | pink
start->connect_rtsp(right)->extract_images(right)->convert_images(right)->send_data(right)->display_data->end
```

相关的代码如下，采用了基于[**Bytedeco**](https://bytedeco.org/)提供的类库以[**FFmpeg**](https://ffmpeg.org/)的形式对视频进行处理。

```java
private void sendImageByRTSP() {
    FFmpegFrameGrabber grabber = null;
    try {
        grabber = FFmpegFrameGrabber.createDefault(rtsp);
        grabber.setOption("rtsp_transport", "tcp"); // 使用tcp的方式，不然会丢包很严重
        grabber.setOption("stimeout", "500000");
        //设置缓存大小，提高画质、减少卡顿花屏
        grabber.setOption("buffer_size", "1024000");
        grabber.start();

        Java2DFrameConverter converter = new Java2DFrameConverter();
        int frequency = 2;
        Frame frame;
        long startTime = System.currentTimeMillis();
        long frameCount = 0;
        while ((frame = grabber.grabImage()) != null) {
            // 按照指定频率处理帧
            if ((System.currentTimeMillis() - startTime) < (frameCount * 1000 / frequency)) {
                continue;
            }

            CompressedImage compressedImage = new CompressedImage();
            Timestamp timestamp = new Timestamp();
            int nano = Instant.now().getNano();
            long second = Instant.now().getEpochSecond();
            timestamp.setSec((int) second);
            timestamp.setNsec(nano);

            BufferedImage image = converter.getBufferedImage(frame);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ImageIO.write(image, "jpeg", baos);

            byte[] encode = Base64.getEncoder().encode(baos.toByteArray());
            String data = new String(encode);
            compressedImage.setData(data);
            compressedImage.setFormat("jpeg");
            compressedImage.setFrameId("main");
            compressedImage.setTimestamp(timestamp);

            JSONObject jsonObject = (JSONObject) JSON.toJSON(compressedImage);
            byte[] bytes = getFormatedBytes(jsonObject.toJSONString().getBytes(), compressedImage.getTimestamp().getNsec(), index);
            this.session.sendBinary(bytes);

            // long类型不用担心溢出
            frameCount++;

            if (frameCount % 100 == 0) {
                log.info(LocalDateTime.now() + "----------------持续发送RTSP视频-------------------");
            }
        }

    } catch (FFmpegFrameGrabber.Exception e) {
        throw new RuntimeException(e);
    } catch (IOException e) {
        throw new RuntimeException(e);
    } finally {
        try {
            grabber.stop();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

上述代码的逻辑虽然不复杂，但在实际使用时却发现播放效果很不流畅，详细定位后发现下述代码耗时较多

```java
ImageIO.write(image, "jpeg", baos); // 此行单次执行需要60-70ms
```

表面上看60ms-70ms耗时不算很多，但若加上后续的格式转化、数据发送、数据播放等阶段，单帧图片在整合流程中的耗时就比较多了，导致实际播放时会有肉眼可见的明显卡顿。



上述代码耗时的原因是由于涉及到IO操作，一开始自己想寻找更高效的替代实现，Google好久之后没有找到更合适的方案，为了满足实际使用需求，此问题又不得不解决。

看来只能另想它法了。

由于`RTSP`视频流可以在浏览器中直接播放，若仿照下图省略掉抽帧、转换、发送等流程，在`Foxglove`中通过浏览器直接播放，岂不会节省很多时间？

```flow
start=>start: 开始
end=>end: 结束
connect_rtsp=>operation: 连接RTSP视频流 | pink
display_data=>operation: UI端展示 | pink
start(right)->connect_rtsp(right)->display_data(right)->end
```

由于`Foxglove`中尚没有类似的面板可直接使用，若要实现类似功能，只能自己开发相关插件，在此之前还需进行可行性验证。

## 初步验证

由于`Foxglove`的播放工作主要是UI端实现的，首先需要验证能否在`HTML`页面中直接播放`RTSP`视频流。

1. 为了模拟`RTSP`视频流，首先需确保安装有`ffmpeg`和[**mediamtx**](https://github.com/bluenviron/mediamtx)环境，首先在命令行中启动`mediamtx`，其输出界面类似如下

   ![mediamtx启动界面](/blog_img/web/foxglove-custom-plugin-develop/mediamtx-console-output.png  "mediamtx启动界面")

2. 在命令行中执行类似如下指令

   ```bash
   ffmpeg -re -i E:\foxglove\usa_drive_2.mp4 -vcodec libx264 -acodec aac -f flv rtmp://127.0.0.1:1935/demo1
   ```

   若输出结果类似如下，则表示`RTSP`视频流环境搭建成功

   ![ffmpeg启动界面](/blog_img/web/foxglove-custom-plugin-develop/ffmpeg-console-output.png  "ffmpeg启动界面")

3. 此时在浏览器中输入`http://127.0.0.1:8888/demo1`即可正常播放该`RTSP`视频流

4. 创建一个`HTML`文件，在其中加入如下内容[^2]，之后用浏览器打开该文件，发现视频内容也能正常播放

   ```html
   <iframe src="http://127.0.0.1:8888/demo1" scrolling="no"></iframe>
   ```

5. 至此可行性验证完毕，实际开发插件时只需要模仿步骤4即可。

## 插件开发

`Foxglove`中开发插件的教程请参见[**create-foxglove-extension**](https://github.com/foxglove/create-foxglove-extension)，本章节以`RTSP`播放为例简要展示开发步骤：

1. 首先确保自己本地安装了`nodejs`[^3]和`npm`环境，之后执行类似下述指令，来初始化工程

   ```bash
   npm init foxglove-extension@latest rtsp-player-extension
   cd rtsp-player-extension
   npm install
   ```

2. 若上述指令执行正常，查看其源码结构类似如下，在开发插件时，只需要修改`src`目录下的源码，其它的基本上不用改动

   ![foxglove插件源码结构](/blog_img/web/foxglove-custom-plugin-develop/rtsp-plugin-project-structure.png  "foxglove插件源码结构")

3. 进一步的查看`src`目录，发现其下只有两个文件: `index.ts`和`ExamplePanel.tsx`，各自内容如下

   `index.ts`内容如下：

   ```typescript
   import { ExtensionContext } from "@foxglove/studio";
   import { initExamplePanel } from "./ExamplePanel";
   
   export function activate(extensionContext: ExtensionContext): void {
     extensionContext.registerPanel({ name: "example-panel", initPanel: initExamplePanel });
   }
   ```

   `ExamplePanel.tsx`内容如下:

   ```typescript
   import { Immutable, MessageEvent, PanelExtensionContext, Topic } from "@foxglove/studio";
   import { useEffect, useLayoutEffect, useState } from "react";
   import ReactDOM from "react-dom";
   
   function ExamplePanel({ context }: { context: PanelExtensionContext }): JSX.Element {
     const [topics, setTopics] = useState<undefined | Immutable<Topic[]>>();
     const [messages, setMessages] = useState<undefined | Immutable<MessageEvent[]>>();
   
     const [renderDone, setRenderDone] = useState<(() => void) | undefined>();
   
     // We use a layout effect to setup render handling for our panel. We also setup some topic subscriptions.
     useLayoutEffect(() => {
       // The render handler is run by the broader studio system during playback when your panel
       // needs to render because the fields it is watching have changed. How you handle rendering depends on your framework.
       // You can only setup one render handler - usually early on in setting up your panel.
       //
       // Without a render handler your panel will never receive updates.
       //
       // The render handler could be invoked as often as 60hz during playback if fields are changing often.
       context.onRender = (renderState, done) => {
         // render functions receive a _done_ callback. You MUST call this callback to indicate your panel has finished rendering.
         // Your panel will not receive another render callback until _done_ is called from a prior render. If your panel is not done
         // rendering before the next render call, studio shows a notification to the user that your panel is delayed.
         //
         // Set the done callback into a state variable to trigger a re-render.
         setRenderDone(() => done);
   
         // We may have new topics - since we are also watching for messages in the current frame, topics may not have changed
         // It is up to you to determine the correct action when state has not changed.
         setTopics(renderState.topics);
   
         // currentFrame has messages on subscribed topics since the last render call
         setMessages(renderState.currentFrame);
       };
   
       // After adding a render handler, you must indicate which fields from RenderState will trigger updates.
       // If you do not watch any fields then your panel will never render since the panel context will assume you do not want any updates.
   
       // tell the panel context that we care about any update to the _topic_ field of RenderState
       context.watch("topics");
   
       // tell the panel context we want messages for the current frame for topics we've subscribed to
       // This corresponds to the _currentFrame_ field of render state.
       context.watch("currentFrame");
   
       // subscribe to some topics, you could do this within other effects, based on input fields, etc
       // Once you subscribe to topics, currentFrame will contain message events from those topics (assuming there are messages).
       context.subscribe([{ topic: "/some/topic" }]);
     }, [context]);
   
     // invoke the done callback once the render is complete
     useEffect(() => {
       renderDone?.();
     }, [renderDone]);
   
     return (
       <div style={{ padding: "1rem" }}>
         <h2>Welcome to your new extension panel!</h2>
         <p>
           Check the{" "}
           <a href="https://foxglove.dev/docs/studio/extensions/getting-started">documentation</a> for
           more details on building extension panels for Foxglove Studio.
         </p>
         <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", rowGap: "0.2rem" }}>
           <b style={{ borderBottom: "1px solid" }}>Topic</b>
           <b style={{ borderBottom: "1px solid" }}>Datatype</b>
           {(topics ?? []).map((topic) => (
             <>
               <div key={topic.name}>{topic.name}</div>
               <div key={topic.datatype}>{topic.datatype}</div>
             </>
           ))}
         </div>
         <div>{messages?.length}</div>
       </div>
     );
   }
   
   export function initExamplePanel(context: PanelExtensionContext): () => void {
     ReactDOM.render(<ExamplePanel context={context} />, context.panelElement);
   
     // Return a function to run when the panel is removed
     return () => {
       ReactDOM.unmountComponentAtNode(context.panelElement);
     };
   }
   ```

   可以看出`index.ts`的作用为注册要开发的面板，而`ExamplePanel.tsx`则是具体业务逻辑实现，同时该文件中有详细的注释辅助我们理解。

   同时通过`ExamplePanel.tsx`也可发现 **`Foxglove`中的插件开发本质上是开发新的面板。**

4. 将`ExamplePanel.tsx`修改为`RTSPPanel.tsx`，并将其内容修改如下：

   ```typescript
   import { Immutable, MessageEvent, PanelExtensionContext} from "@foxglove/studio";
   import { useEffect, useLayoutEffect, useState } from "react";
   import ReactDOM from "react-dom";
   
   interface Timestap{
     sec:number;
     nsec:number;
   }
   
   interface RtspInfo {
     rtsp_url: string;
     chassis_code: string;
     timestamp: Timestap;
   }
   
   const frameStyle = {
     overflow:'hidden',
     height:'100%',
     width:'100%',
   };
   
   function RTSPPanel({ context }: { context: PanelExtensionContext }): JSX.Element {
     const [messages, setMessages] = useState<undefined | Immutable<MessageEvent[]>>();
     const [rtspUrl, setRtspUrl] = useState<string>();
   
     const [renderDone, setRenderDone] = useState<(() => void) | undefined>();
   
     useLayoutEffect(() => {
   
       context.onRender = (renderState, done) => {
   
         setRenderDone(() => done);
         setMessages(renderState.currentFrame);
       };
   
       context.watch("currentFrame");
       context.subscribe([{ topic: "/drive/chassis_code" }]);
     }, [context]);
   
     let rtsp_url:string = 'http://127.0.0.1:8888/demo1';
     useEffect(() => {
       if (messages) {
           messages.forEach(m => {
              let info = m.message as RtspInfo;
              rtsp_url = info.rtsp_url;
              setRtspUrl(rtsp_url)
           })
       }
     }, [messages]);
   
     // invoke the done callback once the render is complete
     useEffect(() => {
       renderDone?.();
     }, [renderDone]);
   
     return (
       <iframe     
          style={ frameStyle }
          src = { rtspUrl as string } 
          allow="autoplay">
       </iframe>
     );
   }
   
   export function initRTSPPanel(context: PanelExtensionContext): () => void {
     ReactDOM.render(<RTSPPanel context={context} />, context.panelElement);
   
     // Return a function to run when the panel is removed
     return () => {
       ReactDOM.unmountComponentAtNode(context.panelElement);
     };
   }
   ```

5. 将`index.ts`修改如下，至此源码修改完成

   ```typescript
   import { ExtensionContext } from "@foxglove/studio";
   import { initRTSPPanel } from "./RTSPPanel";
   
   export function activate(extensionContext: ExtensionContext): void {
     extensionContext.registerPanel({ name: "RTSP流媒体播放", initPanel: initRTSPPanel });
   }
   ```

6. 在项目根目录下打开命令行，执行下述指令，即可生成一个扩展名为`foxe`的插件

   ```bash
   npm run package
   ```

7. 默认情况下生成的插件名类似`unknown.rtsp-player-extension-0.0.0.foxe`，里面的`unknown`和`0.0.0`看着很刺眼，可将`package.json`中的`publisher`和`version`修改为类似如下

   ```java
   {
     "name": "rtsp-player-extension",
     "displayName": "rtsp-player-extension",
     "description": "",
     "publisher": "lucumt",
     "homepage": "",
     "version": "0.0.1",
     "license": "UNLICENSED",
     "main": "./dist/extension.js",
     "keywords": [],
     "scripts": {
       "build": "foxglove-extension build",
       "foxglove:prepublish": "foxglove-extension build --mode production",
       "lint:ci": "eslint --report-unused-disable-directives .",
       "lint": "eslint --report-unused-disable-directives --fix .",
       "local-install": "foxglove-extension install",
       "package": "foxglove-extension package",
       "pretest": "foxglove-extension pretest"
     },
     "devDependencies": {
       // xxx
     }
   }
   ```

   之后重新执行`npm run package`即可生成一个名为`lucumt.rtsp-player-extension-0.0.1.foxe`的插件产物，至此整个插件开发与构建流程结束，接下来的就是安装与使用。

## 安装&使用

1. 在浏览器打开`Foxglove`播放界面，之后打开前述步骤中生成的插件，将其拖动到浏览器窗口中，类似下图所示

   ![foxglove插件安装过程](/blog_img/web/foxglove-custom-plugin-develop/foxglove-plugin-install.png  "foxglove插件安装过程")

2. 若安装成功，会在浏览器顶部出现相关提示

   ![foxglove插件安装成功](/blog_img/web/foxglove-custom-plugin-develop/foxglove-plugin-install-result.png  "foxglove插件安装成功")

3. 在新建面板时，可发现安装的插件位于列表底部，同时其名称中包含 **[local]** 后缀，可仿照正常面板的使用流程来使用该面板，使用效果参见[**RTSP视频播放**](/post/web/foxglove-function-instructions/#图片视频)，至此，`Foxglove` 自定义插件开发与使用的全部流程操作完毕！

   ![foxglove显示自定义开发的面板](/blog_img/web/foxglove-custom-plugin-develop/foxglove-custom-panel.png  "foxglove显示自定义开发的面板")

[^1]: `GitHub`上有人讨论此问题[AVC / H.264 video support #88](https://github.com/orgs/foxglove/discussions/88)，但截止到目前依旧没有整合进去
[^2]: 有关`RTSP`与`HTML`、`JavaScript`整合的说明可参考[https://www.cnblogs.com/badaoliumangqizhi/p/17211019.html](https://www.cnblogs.com/badaoliumangqizhi/p/17211019.html)
[^3]: `nodejs`的版本推荐为`16.xx.xx`，个人实际测试发现用`12.xx.xx`版本时会导致初始化过程出错
[^4]: 此种方式下开发的插件安装后只能与当前浏览器绑定，即相关的浏览器都需要手工安装，若不想重复手工安装，可通过修改`Foxglove`UI端的源码然后重新打包构建镜像。