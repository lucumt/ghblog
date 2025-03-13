---
title: "利用draco的编码解码实现点云数据的高效传输"
date: 2025-02-26T10:20:20+08:00
lastmod: 2025-02-26T10:20:20+08:00
draft: true
keywords: ["点云","three.js","draco","编码","解码"]
description: "基于自己项目中的经验，介绍如何利用Google Draco对点云数据编码与解码，以便减少数据大小加快传输速度"
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
borderImage: false

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

![点云文件传输方式对比](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/ways-to-transmit-point-cloud-data.png "点云文件传输方式对比") 

## 点云编码

```js
'use_strict';

const fs = require('fs');
const draco3d = require('draco3d');
const readline = require('readline');
const {
    styleText
} = require('node:util');

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

分别执行下述指令对3个不同的点云`ply`文件编码为`drc`文件[^1]

```bash
node draco_encode_test.js 000000.ply 000000.drc
node draco_encode_test.js 000001.ply 000001.drc
node draco_encode_test.js 000002.ply 000002.drc
```

执行结果类似如下

![Draco编码结果展示](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-decode-results.png "Draco编码结果展示") 

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

[^1]: 为了便于处理，实际测试时`ply`文件头部的声明标识去掉了只留下了纯坐标数字