---
title: "Foxglove自定义插件开发说明"
date: 2023-11-22T14:08:39+08:00
lastmod: 2023-11-22T14:08:39+08:00
draft: true
keywords: ["foxglove","插件开发"]
description: "Foxglove自定义插件开发说明"
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

# 问题背景

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

# 初步验证

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

4. 创建一个`HTML`文件，在其中加入如下内容，之后用浏览器打开该文件，发现视频内容也能正常播放

   ```html
   <iframe src="http://127.0.0.1:8888/demo1" scrolling="no"></iframe>
   ```

5. 至此可行性验证完毕，实际开发插件时只需要模仿步骤4即可。

# 插件开发

# 安装&使用

[^1]: `GitHub`上有人讨论此问题[AVC / H.264 video support #88](https://github.com/orgs/foxglove/discussions/88)，但截止到目前依旧没有整合进去