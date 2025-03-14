---
title: "利用draco的编码解码实现点云数据的高效传输"
date: 2025-02-26T10:20:20+08:00
lastmod: 2025-02-26T10:20:20+08:00
draft: true
keywords: ["点云","three.js","draco","编码","解码","网络传输"]
description: "基于自己项目中的经验，介绍如何利用Google Draco对点云数据编码与解码，以便缩小点云数据体积实现高效的网络传输"
tags: ["pointcloud","draco"]
categories: ["web编程"]
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
---

个人在项目中使用了[**Draco**](https://github.com/google/draco)对[**点云**](https://en.wikipedia.org/wiki/Point_cloud)数据进行编码与解码，其通过压缩点云数据以减少在网络传输过程中的数据包大小，最终加快点云帧的播放速度，同时由于网络上关于此方面的资料太少，故将其用法和个人踩过的坑简单记录下。

<!--more-->

## 背景

项目中某个功能模块需要将点云文件按帧进行连续播放，实现类似动画播放的效果，但实际对相关功能进行测试时发现有明显延迟，即使在公司内部网络具有`GPU`的环境进行测试也是如此[^1]

![原始点云文件传输耗时](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/original-point-cloud-transmit-time-cost.png "原始点云文件传输耗时") 

从数据截图可看出，即使在同一个区域的公司内部网络，其网络请求耗时也接近1s，而在不同区域的公司内网其耗时已经接近4s，完全不满足使用需求。

造成点云数据传输缓慢的原因是多方面的，如网络环境、数据包体积、服务器性能、浏览器端硬件性能等。

在此问题中采用排除法以及基于前述的截图可以很快的找到问题原因：**网络传输太慢是由于数据包体积太大**，而个人项目中数据包的体积为1.3M，远超正常的网络请求数据包大小。

![Draco对比说明](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-comparing-instruction.png "Draco对比说明") 

要解决

相关的流程如下

![原始点云传输方式](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/original-transmit-point-cloud-data.png "原始点云传输方式") 

整合`Draco`后的点云传输流程如下

![Draco点云传输方式](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-transmit-point-cloud-data.png "Draco点云传输方式") 

改进后的测试结果类似如下，压缩后的体积变为原来的三分之一(与`Draco`官方宣称的压缩率差别较大的原因是自己采用了多帧数据一起传输以及保留了较高的数据精度)，至于网络传输耗时则最快能缩短到100ms之内，即使在不同地域的网络进行点云传输，其耗时相对于改进前也大大缩短，基本上能满足实际生产使用要求。

![Draco点云文件传输耗时](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-point-cloud-transmit-time-cost.png "Draco点云文件传输耗时") 

## 点云编码

```js
'use_strict';

const fs = require('fs');
const draco3d = require('draco3d');
const readline = require('readline');
const { styleText } = require('node:util');

// Global encoder module variables.
let encoderModule = null;
let fileSize = 0,
    encodedSize = 0,
    startTime = null,
    endTime = null;

draco3d.createEncoderModule({}).then(function(module) {
    encoderModule = module;
    console.log('Encoder Module Initialized!');
    if (!encoderModule) {
        return;
    }
    encodeData(process.argv[2], process.argv[3]);
});

function encodeData(srcFile, dstFile) {	
    startTime = new Date();
    let data = [];
    let rl = readline.createInterface({
        input: fs.createReadStream(srcFile),
        crlfDelay: Infinity
    });

    rl.on('line', (line) => {
        data.push(line);
		const encoder = new TextEncoder();
		fileSize += Buffer.byteLength(line);
    });

    rl.on('close', () => {
		console.log("Reading file of size " + styleText('green', `${fileSize}`) + " bytes for file " + srcFile);
		let points = []
		for (i in data) {
			points = [...points, ...data[i].split(" ").map(Number)];
		}
		encodePointCloudToFile(dstFile, points);
    });
}

function encodePointCloudToFile(file, data) {
    const encoder = new encoderModule.Encoder();
    const pointBuilder = new encoderModule.PointCloudBuilder();
    const pointCloud = new encoderModule.PointCloud();

    const attrs = {
        POSITION: 3
    };

    Object.keys(attrs).forEach((attr) => {

        const numValues = data.length;
        const stride = attrs[attr];
        const numPoints = numValues / stride;
        const encoderAttr = encoderModule[attr];

        const attributeDataArray = new Float32Array(numValues);
        for (let i = 0; i < numValues; ++i) {
            attributeDataArray[i] = data[i]
        }

        pointBuilder.AddFloatAttribute(pointCloud, encoderAttr, numPoints, stride, attributeDataArray);
    });


    let encodedData = new encoderModule.DracoInt8Array();
    // Set encoding options.
    encoder.SetSpeedOptions(5, 5);
    encoder.SetAttributeQuantization(encoderModule.POSITION, 10);
    encoder.SetEncodingMethod(encoderModule.MESH_EDGEBREAKER_ENCODING);

    // Encoding.
    console.log("Encoding...");
    encodedSize = encoder.EncodePointCloudToDracoBuffer(pointCloud, false, encodedData);
    encoderModule.destroy(pointCloud);

    if (encodedSize > 0) {
        console.log("Encoded size is " + styleText('green', `${encodedSize}`) + " bytes");
    } else {
        console.log("Error: Encoding failed.");
        return
    }
    // Copy encoded data to buffer.
    const outputBuffer = new ArrayBuffer(encodedSize);
    const outputData = new Int8Array(outputBuffer);
    for (let i = 0; i < encodedSize; ++i) {
        outputData[i] = encodedData.GetValue(i);
    }
    encoderModule.destroy(encodedData);
    encoderModule.destroy(encoder);
    encoderModule.destroy(pointBuilder);
    fs.writeFile(file, Buffer.from(outputBuffer), "binary",
        function(err) {
            if (err) {
                console.log(err);
            } else {
                console.log("The file " + file + " was saved!");
            }
        });
    let endTime = new Date();
    let timeCost = styleText('green', `${endTime - startTime}`)
    let rate = styleText('green', (encodedSize / fileSize * 100).toFixed(2) + '%');
    console.log(`Encode finished,time cost: ${timeCost}ms, compress rate: ${rate}`);
}
```

分别执行下述指令对3个不同的点云`ply`文件编码为`drc`文件[^2]

```bash
node draco_encode_test.js 000000.ply 000000.drc
node draco_encode_test.js 000001.ply 000001.drc
node draco_encode_test.js 000002.ply 000002.drc
```

执行结果类似如下

![Draco编码结果展示](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-encode-results.png "Draco编码结果展示") 

基于上述文件可得出如下结论：

1. 点云数据量较小时，压缩比不高
2. 点云文件越大，压缩耗时越高

## 点云解码

```js
'use_strict';

const fs = require('fs');
const assert = require('assert');
const draco3d = require('draco3d');

// Global decoder module variables.
let decoderModule = null;

draco3d.createDecoderModule({}).then(function(module) {
    decoderModule = module;
    console.log('Decoder Module Initialized!');
    decodeData('000000.drc');
});

function decodeData(srcFile) {
    if (!decoderModule) {
        return;
    }
    fs.readFile(srcFile, function(err, data) {
        if (err) {
            return console.log(err);
        }
        console.log("Decoding file of size " + data.byteLength + " ..");
        // Decode mesh
        const decoder = new decoderModule.Decoder();
        decodeDracoData(data, decoder);
    });
}

function decodeDracoData(rawBuffer, decoder) {
    const buffer = new decoderModule.DecoderBuffer();
    buffer.Init(new Float32Array(rawBuffer), rawBuffer.byteLength);
    const geometryType = decoder.GetEncodedGeometryType(buffer);

    let dracoGeometry = null;
    let status;
    if (geometryType === decoderModule.TRIANGULAR_MESH) {
        dracoGeometry = new decoderModule.Mesh();
        status = decoder.DecodeBufferToMesh(buffer, dracoGeometry);
    } else if (geometryType === decoderModule.POINT_CLOUD) {
        dracoGeometry = new decoderModule.PointCloud();
        status = decoder.DecodeBufferToPointCloud(buffer, dracoGeometry);
    } else {
        const errorMsg = 'Error: Unknown geometry type.';
        console.error(errorMsg);
    }
    console.log(`----------status: ${status}`);
    decoderModule.destroy(buffer);

    const attrs = {
        POSITION: 3
    };
    const numPoints = dracoGeometry.num_points();
    Object.keys(attrs).forEach((attr) => {
        const decoderAttr = decoderModule[attr];
        const attrId = decoder.GetAttributeId(dracoGeometry, decoderAttr);
        const stride = attrs[attr];
        const numValues = numPoints * stride;
        console.log(`----------attrId: ${attrId}`);
        console.log(`----------stride: ${stride}`);
        console.log(`----------numValues: ${numValues}`);

        const attribute = decoder.GetAttribute(dracoGeometry, attrId);
        const attributeData = new decoderModule.DracoFloat32Array();
        decoder.GetAttributeFloatForAllPoints(dracoGeometry, attribute, attributeData);
        let points = [];
        for (let i = 0; i < numValues; i = i + stride) {
            for (let j = i; j < i + stride; j++) {
                points.push(attributeData.GetValue(j));
            }
        }
        console.log(points);
        decoderModule.destroy(attributeData);
    });
}
```

## 精确度丢失

利用下述代码对编码和解码过程中的数据进行输出对比

```js
function printPointClouds(points) {
    // 最多只输出20点的坐标数据，便于进行对比
    let num = points.length < 20 ? points.length : 20;
    for (let i = 0; i < num * 3; i = i + 3) {
        console.log(styleText('yellow', points[i] + '\t\t' + points[i + 1] + '\t' + points[i + 2]));
    }
}
```

## 不同压缩比

原始点云数据直接渲染：

![原始点云渲染](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/original-point-cloud-render.png "原始点云渲染") 

采用`Draco`基于`Float`类型编码解码后渲染：

![基于Float的Draco点云渲染](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/float-draco-point-cloud-render.png "基于Float的Draco点云渲染") 

采用`Draco`基于`Int`类型编码解码后渲染：
![基于Int的Draco点云渲染](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/float-draco-point-cloud-render.png "基于Int的Draco点云渲染") 

可得出如下结论

> **压缩比越高，数据精度越低，耗时也越低**

其本质上是时间换空间。

[^1]: 数据差异较大的原因网络环境导致，左侧为公司内部网络测试、右侧为其它区域的分公司测试
[^2]: 为了便于处理，实际测试时`ply`文件头部的声明标识去掉了只留下了纯坐标数字