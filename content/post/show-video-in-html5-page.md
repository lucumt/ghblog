+++
author = "飞狐"
categories = ["Web编程"]
tags = ["html5"]
date = "2016-10-30T19:48:17+08:00"
description = "Blog of Rosen Lu"
keywords = ["html5"]
title = "在iframe嵌套的html5中播放视频时全屏显示和取消音量调整"

+++


由于项目需要，最近使用了在`html5`中播放视频的功能，期间遇到了几个坑，先简单记录下。

<!--more-->

## 在html5页面中播放视频
如何在`html5`页面中嵌入视频的代码在网上很容易直接搜索到，典型的代码如下所示：

``` html
<video width="320" height="240" controls="controls">
  <source src="movie.mp4" type="video/mp4"/>
  Your browser does not support the video tag.
</video>
```
之后的效果显示如下，从图中我们可以看出该视频播放界面包含 *快进* 、 *音量调整* 和 *全屏播放* 这几个按钮  
![浏览器中正常播放视频](https://ooo.0o0.ooo/2016/10/30/5815e96382a3c.png "浏览器中正常播放视频")

## 在iframe中不能全屏播放视频
项目中好多地方都用`iframe`来嵌套html页面，最开始我是用类似如下代码在被`iframe`包含的页面中嵌入前面的视频播放代码，
发现显示出来的视频播放器没有全屏播放按钮，通过升级浏览器版本和清除缓存等方法依然不奏效。搜索**[stackoverflow](http://stackoverflow.com/)**找到一个类似的问题[How to make a video fullscreen when it is placed inside an iframe?](http://stackoverflow.com/questions/15276929/how-to-make-a-video-fullscreen-when-it-is-placed-inside-an-iframe)，阅读后发现只需要将`iframe`修改成`<iframe … allowfullscreen="true" webkitallowfullscreen="true" mozallowfullscreen="true">`即可
``` html
<iframe>
  <!DOCTYPE html>
  <html>
    <head />
    <body>
      <video width="320" height="240" controls="controls">
		<source src="movie.mp4" type="video/mp4"/>
		Your browser does not support the video tag.
	  </video>
    <body>
  </html>
</iframe>
```  
![视频无法进行全屏播放](https://ooo.0o0.ooo/2016/10/30/581600e124b51.png "视频无法进行全屏播放")

## 隐藏声音调整按钮
有些演示视频只有图像没有声音，为了避免对使用者造成不必要的干扰，可以将声音播放按钮屏蔽掉，由于自己项目只支持基于`webkit`内核的`Chrome`浏览器访问，通过Google之后在 **[stackoverflow](http://stackoverflow.com/)**找到
[Why do no user-agents implement the CSS cursor style for video elements](http://stackoverflow.com/questions/15126921/why-do-no-user-agents-implement-the-css-cursor-style-for-video-elements/15145555#15145555)这篇文章，其中列出了播放视频时相关控制按钮的css类：

- video::-webkit-media-controls-panel
- video::-webkit-media-controls-play-button
- video::-webkit-media-controls-volume-slider-container
- video::-webkit-media-controls-volume-slider
- video::-webkit-media-controls-mute-button
- video::-webkit-media-controls-timeline
- video::-webkit-media-controls-current-time-display
- video::-webkit-full-page-media::-webkit-media-controls-panel
- video::-webkit-media-controls-timeline-container
- video::-webkit-media-controls-time-remaining-display
- video::-webkit-media-controls-seek-back-button
- video::-webkit-media-controls-seek-forward-button
- video::-webkit-media-controls-fullscreen-button
- video::-webkit-media-controls-rewind-button
- video::-webkit-media-controls-return-to-realtime-button
- video::-webkit-media-controls-toggle-closed-captions-button

为了屏蔽掉声音播放按钮，我们只需使用 *video::-webkit-media-controls-volume-slider* 和  *video::-webkit-media-controls-mute-button* 这两个属性即可，相应的css代码如下：  
``` css
/**隐藏视频音量大小调整控件**/
.no_sound_style>video::-webkit-media-controls-volume-slider{
	display:none;
}

/**隐藏视频音量喇叭**/
.no_sound_style>video::-webkit-media-controls-mute-button{
	display:none;
}
```  
对应的显示效果如下图所示，可以看到音量喇叭和音量调整空间都消失不见  
![屏蔽了声音播放按钮](https://ooo.0o0.ooo/2016/10/30/5815f41099feb.png "屏蔽了播放器中的声音播放按钮")