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

简要记录个人使用[**Draco**](https://github.com/google/draco)对[**点云**](https://en.wikipedia.org/wiki/Point_cloud)数据进行编码与解码，减少网络传输过程中的数据体积大小，最终加快网络请求的过程。

<!--more-->

## 背景

## 编码

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

## 解码

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



## 浮点数转换

## 整数转换

## 不同压缩比

