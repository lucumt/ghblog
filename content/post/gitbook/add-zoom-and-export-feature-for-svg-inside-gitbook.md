---
title: "给GitBook中的SVG图像提供缩放和下载功能"
date: 2024-06-27T20:35:45+08:00
lastmod: 2024-06-27T20:35:45+08:00
draft: true
keywords: ["gitbook","svg","缩放","下载"]
description: "简要介绍如何给GitBook中的SVG图像提供缩放和下载功能"
tags: ["gitbook"]
categories: ["工具使用"]
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

简要说明如何给`GitBook`中的`SVG`图像提供缩放和下载功能

<!--more-->

# 背景

# 实现

## 相关代码

* 文件加载

* 核心代码

  ```javascript
  require([
    'gitbook'
  ], function (gitbook) {
    gitbook.events.bind('page.change', function () {
      mermaid.run({
          querySelector: '.mermaid',
          postRenderCallback: (id) => {
  		  let ele = document.getElementById(id);
            let svg = ele.getBBox();
  		  let height = svg.height;
  		  let aHeight = height > 800 ? 800 : height;
  		  ele.setAttribute('style','height: '+aHeight+'px;overflow:scroll;');
  		  let panZoomTiger = svgPanZoom('#'+id,{
  			  zoomEnabled: true,
  			  controlIconsEnabled: true
  		  });
  		  panZoomTiger.resize();
  		  panZoomTiger.updateBBox();
  		  
  		  let download = ele.parentNode.previousSibling;
  		  download.addEventListener('click',e => {
  			  downloadData(id, ele);
  		  });
          }
      });
    });
  });
  
  function downloadData(id,ele) {
  	
  	let svg = ele.cloneNode(true);
	
		// remove svg-pan-zoom-controls for the download file
  	svg.getElementById("svg-pan-zoom-controls").remove();
  	
  	let serializer = new XMLSerializer();
  	let source = serializer.serializeToString(svg);
  	source = source.replace(/(\w+)?:?xlink=/g, 'xmlns:xlink='); // Fix root xlink without namespace
  	source = source.replace(/ns\d+:href/g, 'xlink:href'); // Safari NS namespace fix.
  
  
  	if (!source.match(/^<svg[^>]+xmlns="http\:\/\/www\.w3\.org\/2000\/svg"/)) {
  		source = source.replace(/^<svg/, '<svg xmlns="http://www.w3.org/2000/svg"');
  	}
  	if (!source.match(/^<svg[^>]+"http\:\/\/www\.w3\.org\/1999\/xlink"/)) {
  		source = source.replace(/^<svg/, '<svg xmlns:xlink="http://www.w3.org/1999/xlink"');
  	}
  
  	let preface = '<?xml version="1.0" standalone="no"?>\r\n';
  	let svgBlob = new Blob([preface, source], { type: "image/svg+xml;charset=utf-8" });
  	let svgUrl = URL.createObjectURL(svgBlob);
  	let downloadLink = document.createElement("a");
  	let name = id + '.svg';
  	
  	downloadLink.download = name;
  	downloadLink.href = svgUrl;
  	document.body.appendChild(downloadLink);
  	downloadLink.click();
  	document.body.removeChild(downloadLink);
  }
  ```

  