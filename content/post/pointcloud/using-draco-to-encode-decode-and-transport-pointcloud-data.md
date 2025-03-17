---
title: "利用Draco对点云数据进行编码解码以实现高效网络传输"
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

要解决此问题也很简单，只需要有针对性的减少网络传输中的数据包大小即可，而之前项目中已经采用了常规的[gzip](https://en.wikipedia.org/wiki/Gzip)与[Protocol Buffers](https://protobuf.dev/)对相关请求进行了压缩处理，必须采用其它方式进一步的减少数据包体积。

![原始点云传输方式](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/original-transmit-point-cloud-data.png "原始点云传输方式") 

## 整合Draco

在常规的数据压缩方式不满足要求之后，只能结合本身的业务特性进一步的寻求有针对性的压缩算法，由于项目中主要是涉及到点云数据，属于三维数据处理的范畴，一番对比后我们选择了`Draco`!

`Draco`是`Google`官方推出专门处理3D数据的开源数据库，在其[官方文档](https://github.com/google/draco)中有如下说明

> Draco is a library for compressing and decompressing 3D geometric [meshes](https://en.wikipedia.org/wiki/Polygon_mesh) and [point clouds](https://en.wikipedia.org/wiki/Point_cloud). It is intended to improve the storage and transmission of 3D graphics.

可看出其主要作用是通过数据压缩与解压，来提升网格和点云数据的存储与传输效率。

其本质上还是通过基于特定业务场景的算法对数据进行针对性的压缩，来减少其大小，数据变小后，当然能存储更多的数据，也能传输的更快！

在[这篇文章](https://opensource.googleblog.com/2017/01/introducing-draco-compression-for-3d.html)中对其压缩比有直观对比展示，相对于常用的`zip`压缩，其压缩比很高，官方文档说明**最高可达到1%**。

![Draco对比说明](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-comparing-instruction.png "Draco对比说明") 

`Draco`主要是采用算法，将点云文件(通常是`ply`文件)或数据压缩编码为`drc`文件，此文件相对于原始的点云文件体积很小，适合网络传输，客户端接收后基于`Draco`进行解码为实际的点云数据[^2]，然后进行播放。

相比之前的流程，整合`Draco`后的改进流程如下：

![Draco点云传输方式](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-transmit-point-cloud-data.png "Draco点云传输方式") 

改进后的测试结果类似如下，压缩后的体积变为原来的三分之一，至于网络传输耗时则最快能缩短到100ms之内，即使在不同地域的网络进行点云传输，其耗时相对于改进前也大大缩短，基本上能满足实际生产使用要求。

![Draco点云文件传输耗时](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-point-cloud-transmit-time-cost.png "Draco点云文件传输耗时") 

上图中采用`Draco`之后的数据压缩率相对于之前只有30%左右，与其官方宣称的最高1%的压缩率差异较大的原因如下：

1. 自己测试中采用的是4帧点云数据同时传输，数据包里面附带了一些其它信息
2. 自己项目中已经对原始点云数据采用`gzip`与`Protocol Buffers`进行了前期的处理，4帧点云原始大小为12M左右
3. 为了保留较高的精确度，适当调整了压缩比设置参数

## 使用说明

`Draco`官方提供如下2种数据编解码操作：

1. 通过编程语言实现，当前主要是`C++`和`JavaScript`，从前面图示中可看出，在相同条件下使用`C++`进行编解码时其耗时相对于`JavaScript`耗时更短，当对性能严苛要求时，优先推荐`C++`实现。

2. 通过脚本实现，主要是编译后的`C++`脚本，以命令行参数的形式执行，可结合`Shell`脚本实现批量的编码与解码

   ```bash
   # 数据编码
   ./draco_encoder -point_cloud -i testdata/bun_zipper.ply -o out.drc
   
   # 数据解码
   ./draco_decoder -i in.drc -o out.obj
   ```

本文主要聚焦于通过使用`JavaScript`以代码的方式对其进行相关操作。



网络上关于`Draco`使用的资料不是很多，自己主要参考的是[draco_nodejs_example.js](https://github.com/google/draco/blob/main/javascript/npm/draco3d/draco_nodejs_example.js)中的相关实现，由于该示例中编解码实现都是基于网格(`Mesh`)实现，而个人项目涉及到的是点云(`PointCloud`)，故需要对其做适当的改进。

理论上只需要将对应方法中的`Mesh`修改为`PointCloud`即可，自己实际操作时发现此路不通，只能基于其官方提供的[IDL](https://en.wikipedia.org/wiki/IDL_specification_language)格式的说明文件[draco_web_encoder.idl](https://github.com/google/draco/blob/main/src/draco/javascript/emscripten/draco_web_encoder.idl)和[draco_web_decoder.idl](https://github.com/google/draco/blob/main/src/draco/javascript/emscripten/draco_web_decoder.idl)进行修改。

`IDL`中的描述类似如下，对于有编程基础的人而言很容易看懂。

```idl
interface PointCloudBuilder {
  void PointCloudBuilder();
  long AddFloatAttribute(PointCloud pc, draco_GeometryAttribute_Type type,
                         long num_vertices, long num_components,
                         [Const] float[] att_values);
  long AddInt8Attribute(PointCloud pc, draco_GeometryAttribute_Type type,
                        long num_vertices, long num_components,
                        [Const] byte[] att_values);
  long AddUInt8Attribute(PointCloud pc, draco_GeometryAttribute_Type type,
                         long num_vertices, long num_components,
                         [Const] octet[] att_values);
                         
    // xxx
};
```

## 点云编码

基于`JavaScript`修改后的`Draco`点云编码实现如下，核心将指定的点云文件转化为指定的`drc`文件。

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
        let readEndTime = new Date();
        let readTimeCost = styleText('green', `${readEndTime - startTime}`);
        console.log("Reading file of size " + styleText('green', `${fileSize}`) + " bytes for file " + srcFile + `,time cost: ${readTimeCost}ms`);
        let points = []
        for (i in data) {
            let pdata = convertPcdToPointCloudData(data[i]);
            if (!!pdata) {
                points = [...points, ...pdata];
            }
        }
        // 计算大小时排除掉表头的声明部分
        let pointSize = styleText('green', `${data.length - 7}`);
        console.log(`point size: ${pointSize}`);
        //printPointClouds(points);
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
    let timeCost = styleText('green', `${endTime - startTime}`);
    let rate = styleText('green', (encodedSize / fileSize * 100).toFixed(2) + '%');
    console.log(`Encode finished,time cost: ${timeCost}ms, compress rate: ${rate}`);
}

function convertPcdToPointCloudData(line) {
    let data = line.split(/\s/);
    if (data.length != 3) {
        return null;
    }
    for (let i = 0; i < 3; i++) {
        if (!isNumeric(data[i])) {
            return null;
        }
    }
    return [Number(data[0]), Number(data[1]), Number(data[2])];
}

function isNumeric(str) {
    if (typeof str != "string") {
        return false;
    }
    return !isNaN(str) && !isNaN(parseFloat(str));
}
```

分别执行下述指令对3个不同的点云`ply`文件编码为`drc`文件

```bash
node draco_encode_test.js 000000.ply 000000.drc
node draco_encode_test.js 000001.ply 000001.drc
node draco_encode_test.js 000002.ply 000002.drc
```

执行结果类似如下

![Draco编码结果展示](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-encode-results.png "Draco编码结果展示") 

基于上述操作可得出如下结论：

1. 点云数据量较小时，压缩率不高，事实上太小的数量也没必要这么折腾
2. 点云文件越大，压缩耗时越高，主要耗时点在于`Draco`自身相关的操作
3. 在文件大小相似时，不同内容的点云文件，其压缩率也可能不相同，但不会偏离太大
4. 鱼与熊掌不可兼得，`Draco`相当于提前利用编码过程中的耗时来实现减小文件体积与缩小传输耗时，另一种时间换空间？

## 点云解码

基于`JavaScript`修改后的`Draco`点云解码实现如下

```js
'use_strict';

const fs = require('fs');
const draco3d = require('draco3d');
const { styleText } = require('node:util');

// Global decoder module variables.
let decoderModule = null;

draco3d.createDecoderModule({}).then(function(module) {
    decoderModule = module;
    console.log('Decoder Module Initialized!');
    decodeData(process.argv[2]);
});

let startTime = null;

function decodeData(srcFile) {
    if (!decoderModule) {
        return;
    }
    startTime = new Date();
    fs.readFile(srcFile, function(err, data) {
        if (err) {
            return console.log(err);
        }
        let fileSize = styleText('green', `${data.byteLength} bytes`);
        console.log("Decoding file of size " + fileSize + " ..");
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

        const attribute = decoder.GetAttribute(dracoGeometry, attrId);
        const attributeData = new decoderModule.DracoFloat32Array();
        decoder.GetAttributeFloatForAllPoints(dracoGeometry, attribute, attributeData);
        let points = [];
        for (let i = 0; i < numValues; i = i + stride) {
            for (let j = i; j < i + stride; j++) {
                points.push(attributeData.GetValue(j));
            }
        }
        decoderModule.destroy(attributeData);
    });
    let endTime = new Date();
    let timeCost = styleText('green', `${endTime - startTime}`);
    console.log(`Encode finished,time cost: ${timeCost}ms, decode point size: ` + styleText('green', `${numPoints}`));
    decoderModule.destroy(decoder);
    decoderModule.destroy(dracoGeometry);
}
```

执行下述指令对前述生成的drc文件分别进行解码

```bash
node draco_decode_test.js 000000.drc
node draco_decode_test.js 000001.drc
node draco_decode_test.js 000002.drc
```

执行结果类似如下，可以看出虽然其解码耗时也随着`drc`变大而增多，但相对于`JavaScript`形式的编码而言已经非常短，可直接用于生产环境。

![Draco解码结果展示](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-decode-results.png "Draco解码结果展示") 

进一步同前述编码过程的输出对比，可发现编码解码后的点云总数保持一致，那为啥其文件体积能缩小这么多呢？

除了编码算法的功劳，还在于**其在不影响使用的前提下牺牲了一部分的精确度**，解码后的数据值无法完全同编码前的保持一致。

![Draco编码解码结果对比](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-encode-decode-compare.png "Draco编码解码结果对比") 

## 脚本转换

前述过程中演示了基于`js`代码实现的编码与解码，虽然解码过程很快，但是编码过程很慢，实际使用中不可能忍受如此长的编码时间。

前述的官方对比中采用`C++`进行编解码效率更高，故可采用基于`C++`编译后的脚本进行数据编解码来进一度缩短耗时。

1.在`Draco`官网下载对应的源码文件后解压，进入`cmake`目录下，可发现有相关的编译文件

![Draco编译文件列表](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-cmake-files.png "Draco编译文件列表") 

2.基于[此说明](https://github.com/google/draco/blob/main/BUILDING.md)在`Draco`的根目录下执行如下操作

```bash
mkdir build_dir && cd build_dir
cmake ../
```

3.若是初次执行，可能会有如下报错

![Draco CMake失败](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-cmake-error.png "Draco CMake失败") 

4.根据不同的操作系统，需要查询`no cmake_cxx_compiler could be found`的解决方案，并进行针对性的修复，以`CentOS 9`为例，可执行如下指令

```bash
yum -y update
yum -y install g++
```

5.之后重新重新步骤2中的编译指令，可正常执行，同时可发现在当前目录下生成了一个名为`Makefile`的文件

![Draco CMake成功](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-cmake-success.png "Draco CMake成功") 

6.在当前目录下执行`make`指令

![Draco开始make编译](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-make-progress.png "Draco开始make编译") 

7.若一切正常，`make`指令编译后的输出类似如下，其中标红的即为可供最终使用的编码与解码脚本

![Draco中make编译结果](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-make-result.png "Draco中make编译结果") 

8.执行下述指令，创建对应的测试目录，并将`ply`文件拷贝到`ply_src_data`目录下去

```bash
cd .. && mkdir -p draco_test/ply_src_data

cp build_dir/draco_encoder draco_test/
cp build_dir/draco_decoder draco_test/

cd draco_test

# 将ply文件拷贝到ply_src_data目录下去
```

9.`Draco`官方提供的编码指令类似

```bash
# 包含网格数据
./draco_encoder -i testdata/bun_zipper.ply -o out.drc

# 单纯的对点云进行编码
./draco_encoder -point_cloud -i testdata/bun_zipper.ply -o out.drc

# 采用更高的压缩率
./draco_encoder -point_cloud -cl 10 -i testdata/bun_zipper.ply -o out.drc
```

为了便于批量测试与统计，可将其封装为类似如下的`Shell`脚本

```bash
#!/bin/sh

encode_drc(){
  file=$1
  dst_folder=$2
  filename1=$(echo $file | awk -F "/" '{print $NF}')
  filename2=$(echo $filename1 | awk -F "." '{print $1}')

  echo "--------------begin to encode ${filename2}-------------------"

  # 记录开始时间（秒.纳秒）
  start=$(date +%s.%N)

  # 转化为ply文件
  echo $filename1
  ./draco_encoder -point_cloud -i $file -o ${dst_folder}/${filename2}.drc

  # 记录结束时间
  end=$(date +%s.%N)

  # 计算时间差（保留6位小数，即微秒）
  runtime=$(echo "scale=3; ($end - $start) * 1000" | bc)

  # 输出结果
  GREEN='\033[32m'
  # 重置样式
  RESET='\033[0m'
  echo -e "${filename1} enecode time cost: ${GREEN}${runtime}${RESET} ms\n\n"
}

dst_folder='drc_encode_result'
rm -rf ${dst_folder}
mkdir ${dst_folder}
for file in ply_src_data/*.ply; do
    encode_drc $file ${dst_folder}
done
```

10.编码后测试结果如下，对比可看出，相对于`js`版本的耗时操作，采用脚本的方式几乎都是**秒级完成**，实际生产环境建议采用`C++`代码进行编码或类似上述的脚本编码。

![Draco脚本文件编码结果](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-script-encode-result.png "Draco脚本文件编码结果") 

同时输出过程中也提示可基于类似`-cl 10`的设置来获得更好的压缩效果，此操作虽然能减少文件大小，但同时会牺牲一定程度的精确度，后续部分会说明。

11.`Draco`官方提供的编码脚本相对简单

```bash
# 输出为obj文件
./draco_decoder -i in.drc -o out.obj

# 输出为ply文件
./draco_decoder -i in.drc -o out.ply
```

同样将其封装为`Shell`脚本

```bash
#!/bin/sh

decode_drc(){
  file=$1
  dst_folder=$2
  filename1=$(echo $file | awk -F "/" '{print $NF}')
  filename2=$(echo $filename1 | awk -F "." '{print $1}')

  echo "--------------begin to decode ${filename1}-------------------"

  # 记录开始时间（秒.纳秒）
  start=$(date +%s.%N)

  # 转化为ply文件
  dst_file=${dst_folder}/${filename2}.ply
  ./draco_decoder -i $file -o ${dst_file}

  # 记录结束时间
  end=$(date +%s.%N)

  # 计算时间差（保留6位小数，即微秒）
  runtime=$(echo "scale=3; ($end - $start) * 1000" | bc)

  # 输出结果
  GREEN='\033[32m'
  # 重置样式
  RESET='\033[0m'
  point_size=$(awk 'NR==3' ${dst_file} | awk '{print $NF}')
  echo -e "${filename1} decode time cost: ${GREEN}${runtime}${RESET} ms, point size: ${GREEN}${point_size}${RESET}\n"
}

dst_folder='ply_decode_result'
rm -rf ${dst_folder}
mkdir ${dst_folder}
for file in drc_encode_result/*.drc; do
    decode_drc $file ${dst_folder}
done
```

12.解码操作执行结果如下，可看出解码后的点云数据总数与原始文件中保持一致，由于即使是`js`的解码耗时也不太多，故实际生产中采用脚本解码的方式并不常见。

![Draco脚本文件解码结果](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/draco-script-decode-result.png "Draco脚本文件解码结果") 

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

// todo 不同压缩比实现

## 效果对比

原始点云数据直接渲染的效果如下

![原始点云渲染](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/original-point-cloud-render.png "原始点云渲染") 

采用`Draco`基于`Float`类型编码解码或采用高压缩率后渲染效果如下，如前所述此时其会损失一定程度的精确度，导致渲染后的效果与原始渲染效果有肉眼可见的清晰度差异。

尽管如此还是能看清楚要展示的内容，此种方式适合对性能要求严苛的场景。

![基于Float的Draco点云渲染](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/float-draco-point-cloud-render.png "基于Float的Draco点云渲染") 

采用`Draco`基于`Int`类型编码或者低压缩率后渲染效果如下，其显示效果与原始渲染效果差别已经很接近了。
![基于Int的Draco点云渲染](/blog_img/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/int-draco-point-cloud-render.png "基于Int的Draco点云渲染") 

[^1]: 数据差异较大的原因网络环境导致，左侧为公司内部网络测试、右侧为其它区域的分公司测试
[^2]: 基于编码压缩时设置的参数，实际解码后的数据与原始数据会有一定程度的误差，但整体上不会对正常使用造成影响

