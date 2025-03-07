---
title: "在Nodejs和React中使用Protocol Buffers对数据进行编码解码"
date: 2024-12-20T10:13:13+08:00
lastmod: 2024-12-20T10:13:13+08:00
draft: true
keywords: ["Protocol Buffers","React","Node js"]
description: "基于个人实际使用经验，简要分享下如何在Nodejs和React中使用Protocol Buffers进行编码/解码，从而提高网络传输效率"
tags: ["react","protocol buffers"]
categories: ["web编程"]
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
borderImage: false
codeTabSeperator: "::"
moreMeta: true

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

网络上关于如何在[**React**](https://react.dev/)中使用[**Protocol Buffers**](https://protobuf.dev/)进行数据编码/解码的资料不是很多，正好自己有这方面的需求，参考多方面的资料后将整个流程调试通过，故简单记录下，供自己和他人使用参考。

<!--more-->

## 文件定义

可通过下述指令查看`React`的版本，自己当前的版本为`19.0.0`

```bash
# 简单输出版本信息
npm show react version

# 详细输出版本信息
npm info react
```

假设要从某个服务器上获取一系列的[点云](https://zh.wikipedia.org/zh-cn/%E9%BB%9E%E9%9B%B2)数据进行渲染显示，由于点云数据的体积通常较大，为了缩短网络传输耗时，需对其进行编码与压缩来减少数据体积,

相关流程如下

![Protocol Buffers编码与解码](/blog_img/web/using-protobuf-in-react/protobuf-encode-decode-data.png "Protocol Buffers编码与解码")

可基于`Protocol Buffers`定义如下文件来表示单帧点云数据，其文件名假设为`prc.proto`

```protobuf
syntax = "proto3";

message PcdData{
  // 数据帧序号
  optional int32 idx=1;
  
  // 数据帧名称
  string name=2;
  
  // 具体数据
  repeated float point=3;
}
```

## 数据编码

数据编码通常是在服务器端(或发送端)进行的，通过将相关数据编码为`Protocol Buffers`格式来减少数据传输体积，从而加快其传输效率。

`Protocol Buffers`数据编码过程主要是利用[pbjs](https://www.npmjs.com/package/pbjs)将`proto`规范文件编译为`json`格式，然后填充数据并传输，此部分实现和`React`关系不大。

以前面的`proto`文件为例，相关操作步骤如下：

1.进入对应目录下安装`protobufjs-cli`，安装完毕后可测试`pbjs`是否正常

```bash
npm i protobufjs-cli

# 采用下述指令进行测试
node ./node_modules/protobufjs-cli/bin/pbjs
```

若安装正常，`pbjs`测试结果如下，关于此指令的详细使用，可参见[说明](https://cloud.tencent.com/developer/article/1960058)

![pbjs指令测试](/blog_img/web/using-protobuf-in-react/pbjs-command-test.png "pbjs指令测试")

2.执行下述指令，分别生成对应的`json`规范文件

```bash
node ./node_modules/protobufjs-cli/bin/pbjs prc.proto -o pcd_data.json

# 显示指定格式
node ./node_modules/protobufjs-cli/bin/pbjs prc.proto -t json -o pcd_data.json
```

3.生成完毕的`json`文件如下

```json
{
  "options": {
    "syntax": "proto3"
  },
  "nested": {
    "PcdData": {
      "oneofs": {
        "_idx": {
          "oneof": [
            "idx"
          ]
        }
      },
      "fields": {
        "idx": {
          "type": "int32",
          "id": 1,
          "options": {
            "proto3_optional": true
          }
        },
        "name": {
          "type": "string",
          "id": 2
        },
        "point": {
          "rule": "repeated",
          "type": "float",
          "id": 3
        }
      }
    }
  }
}
```

4.基于此[说明](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs)创建一个简单的服务器端，命名为`server.js`，确保其与前面的文件都处于同一个目录下

```js
const { createServer } = require('node:http');
const hostname = '127.0.0.1';
const port = 3000;
const server = createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World');
});
server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

可通过`node server.js`来启动并测试

5.假设当前目录下有一个名为`000010.pcd`的点云文件，修改`server.js`添加上点云读取写入的相关逻辑，修改后的代码如下

```js{data-line="7-10,37-43"}
const { createServer } = require('node:http');
const hostname = '127.0.0.1';
const port = 3100;

let fs = require('fs');
let readline = require('readline');
let protobufjs = require("protobufjs");
let pcdJson = require("./pcd_data.json")
let pcdRoot = protobufjs.Root.fromJSON(pcdJson)
let pcdMessage = pcdRoot.lookupType("PcdData");

const server = createServer((req, res) => {
  res.statusCode = 200;
  loadPcdBinaryFile(req, res);
});
server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});


async function loadPcdBinaryFile(req, res) {
  let fileStream = fs.createReadStream(`000010.pcd`);
  let fileData = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity
  });

  let points = [];
  for await (const line of fileData) {
    let data = processPcdData(line);
    if (data == null) {
      continue;
    }
    points.push(...data);
  }

  let result = { idx: 1, name: '000010.pcd', point: points }
  let buff = pcdMessage.encode(pcdMessage.create(result)).finish();
  res.setHeader('Content-Type', 'application/octet-stream');
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Headers", "X-Requested-With");
  res.write(buff,'binary');
  res.end(null, 'binary');
}

// 对点云文件进行处理，过滤掉无用的数据
function processPcdData(line) {
  let data = line.split(/\s/)
  if (data.length != 4) {
    return null;
  }
  for (let i = 0; i < 4; i++) {
    if (!isNumeric(data[i])) {
      return null;
    }
  }
  return [Number(data[0]), Number(data[1]), Number(data[2])]
}

function isNumeric(str) {
  if (typeof str != "string") {
    return false;
  }
  return !isNaN(str) && !isNaN(parseFloat(str));
}
```

关键部分代码已经高亮显示，可看出实际编码量并不多。

6.在浏览器中打开`http://127.0.0.1:3100`可正常下载编码后的点云文件，用记事本或其它编辑器打开类似如下，由于`Protocol Buffers`编码的格式无法直接查看，所以大部分内容为乱码，尽管如此，其中的部分属性还是能正常查看(如name属性)。

至此，`Protocol Buffers`数据编码操作完成。

![编码后的点云文件](/blog_img/web/using-protobuf-in-react/pcd-protobuf-file-open.png "编码后的点云文件")

## 数据解码

此部分操作采用`React`进行，相关操作步骤如下：

1.执行下述指令创建一个`React`项目并创建相关的依赖

```bash
npx create-react-app react-client-test -y
cd react-client-test
npm i protobufjs 

# 个人实际操作发现此依赖包必须全局安装，否则会导致步骤3出错
npm i -g protoc-gen-js
```

2.基于此[说明](https://grpc.io/docs/protoc-installation/)安装Protocol Buffer Compiler，安装完毕后可通过下述操作检查其是否正常

![Proctoc版本检查](/blog_img/web/using-protobuf-in-react/protoc-version-check.png "Proctoc版本检查")

3.将前述的`prc.proto`文件拷贝到该项目的`src`目录下，之后执行下述指令生成`Protocol Buffers`对应的反序列化文件

```bash
# 必须在proto文件所在的目录下执行
protoc --proto_path=./ --js_out=import_style=commonjs,binary:. ./*.proto
```

4.若一切正常，则会生成后缀为`_pb.js`的文件，本例中为`prc_pb.js`，其源码如下

{{< details "+点击以展开/折叠" >}} 

```js
// source: prc.proto
/**
 * @fileoverview
 * @enhanceable
 * @suppress {missingRequire} reports error on implicit type usages.
 * @suppress {messageConventions} JS Compiler reports an error if a variable or
 *     field starts with 'MSG_' and isn't a translatable message.
 * @public
 */
// GENERATED CODE -- DO NOT EDIT!
/* eslint-disable */
// @ts-nocheck

var jspb = require('google-protobuf');
var goog = jspb;
var global =
    (typeof globalThis !== 'undefined' && globalThis) ||
    (typeof window !== 'undefined' && window) ||
    (typeof global !== 'undefined' && global) ||
    (typeof self !== 'undefined' && self) ||
    (function () { return this; }).call(null) ||
    Function('return this')();

goog.exportSymbol('proto.PcdData', null, global);
/**
 * Generated by JsPbCodeGenerator.
 * @param {Array=} opt_data Optional initial data array, typically from a
 * server response, or constructed directly in Javascript. The array is used
 * in place and becomes part of the constructed object. It is not cloned.
 * If no data is provided, the constructed object will be empty, but still
 * valid.
 * @extends {jspb.Message}
 * @constructor
 */
proto.PcdData = function(opt_data) {
  jspb.Message.initialize(this, opt_data, 0, -1, proto.PcdData.repeatedFields_, null);
};
goog.inherits(proto.PcdData, jspb.Message);
if (goog.DEBUG && !COMPILED) {
  /**
   * @public
   * @override
   */
  proto.PcdData.displayName = 'proto.PcdData';
}

/**
 * List of repeated fields within this message type.
 * @private {!Array<number>}
 * @const
 */
proto.PcdData.repeatedFields_ = [3];



if (jspb.Message.GENERATE_TO_OBJECT) {
/**
 * Creates an object representation of this proto.
 * Field names that are reserved in JavaScript and will be renamed to pb_name.
 * Optional fields that are not set will be set to undefined.
 * To access a reserved field use, foo.pb_<name>, eg, foo.pb_default.
 * For the list of reserved names please see:
 *     net/proto2/compiler/js/internal/generator.cc#kKeyword.
 * @param {boolean=} opt_includeInstance Deprecated. whether to include the
 *     JSPB instance for transitional soy proto support:
 *     http://goto/soy-param-migration
 * @return {!Object}
 */
proto.PcdData.prototype.toObject = function(opt_includeInstance) {
  return proto.PcdData.toObject(opt_includeInstance, this);
};


/**
 * Static version of the {@see toObject} method.
 * @param {boolean|undefined} includeInstance Deprecated. Whether to include
 *     the JSPB instance for transitional soy proto support:
 *     http://goto/soy-param-migration
 * @param {!proto.PcdData} msg The msg instance to transform.
 * @return {!Object}
 * @suppress {unusedLocalVariables} f is only used for nested messages
 */
proto.PcdData.toObject = function(includeInstance, msg) {
  var f, obj = {
idx: (f = jspb.Message.getField(msg, 1)) == null ? undefined : f,
name: jspb.Message.getFieldWithDefault(msg, 2, ""),
pointList: (f = jspb.Message.getRepeatedFloatingPointField(msg, 3)) == null ? undefined : f
  };

  if (includeInstance) {
    obj.$jspbMessageInstance = msg;
  }
  return obj;
};
}


/**
 * Deserializes binary data (in protobuf wire format).
 * @param {jspb.ByteSource} bytes The bytes to deserialize.
 * @return {!proto.PcdData}
 */
proto.PcdData.deserializeBinary = function(bytes) {
  var reader = new jspb.BinaryReader(bytes);
  var msg = new proto.PcdData;
  return proto.PcdData.deserializeBinaryFromReader(msg, reader);
};


/**
 * Deserializes binary data (in protobuf wire format) from the
 * given reader into the given message object.
 * @param {!proto.PcdData} msg The message object to deserialize into.
 * @param {!jspb.BinaryReader} reader The BinaryReader to use.
 * @return {!proto.PcdData}
 */
proto.PcdData.deserializeBinaryFromReader = function(msg, reader) {
  while (reader.nextField()) {
    if (reader.isEndGroup()) {
      break;
    }
    var field = reader.getFieldNumber();
    switch (field) {
    case 1:
      var value = /** @type {number} */ (reader.readInt32());
      msg.setIdx(value);
      break;
    case 2:
      var value = /** @type {string} */ (reader.readString());
      msg.setName(value);
      break;
    case 3:
      var values = /** @type {!Array<number>} */ (reader.isDelimited() ? reader.readPackedFloat() : [reader.readFloat()]);
      for (var i = 0; i < values.length; i++) {
        msg.addPoint(values[i]);
      }
      break;
    default:
      reader.skipField();
      break;
    }
  }
  return msg;
};


/**
 * Serializes the message to binary data (in protobuf wire format).
 * @return {!Uint8Array}
 */
proto.PcdData.prototype.serializeBinary = function() {
  var writer = new jspb.BinaryWriter();
  proto.PcdData.serializeBinaryToWriter(this, writer);
  return writer.getResultBuffer();
};


/**
 * Serializes the given message to binary data (in protobuf wire
 * format), writing to the given BinaryWriter.
 * @param {!proto.PcdData} message
 * @param {!jspb.BinaryWriter} writer
 * @suppress {unusedLocalVariables} f is only used for nested messages
 */
proto.PcdData.serializeBinaryToWriter = function(message, writer) {
  var f = undefined;
  f = /** @type {number} */ (jspb.Message.getField(message, 1));
  if (f != null) {
    writer.writeInt32(
      1,
      f
    );
  }
  f = message.getName();
  if (f.length > 0) {
    writer.writeString(
      2,
      f
    );
  }
  f = message.getPointList();
  if (f.length > 0) {
    writer.writePackedFloat(
      3,
      f
    );
  }
};


/**
 * optional int32 idx = 1;
 * @return {number}
 */
proto.PcdData.prototype.getIdx = function() {
  return /** @type {number} */ (jspb.Message.getFieldWithDefault(this, 1, 0));
};


/**
 * @param {number} value
 * @return {!proto.PcdData} returns this
 */
proto.PcdData.prototype.setIdx = function(value) {
  return jspb.Message.setField(this, 1, value);
};


/**
 * Clears the field making it undefined.
 * @return {!proto.PcdData} returns this
 */
proto.PcdData.prototype.clearIdx = function() {
  return jspb.Message.setField(this, 1, undefined);
};


/**
 * Returns whether this field is set.
 * @return {boolean}
 */
proto.PcdData.prototype.hasIdx = function() {
  return jspb.Message.getField(this, 1) != null;
};


/**
 * optional string name = 2;
 * @return {string}
 */
proto.PcdData.prototype.getName = function() {
  return /** @type {string} */ (jspb.Message.getFieldWithDefault(this, 2, ""));
};


/**
 * @param {string} value
 * @return {!proto.PcdData} returns this
 */
proto.PcdData.prototype.setName = function(value) {
  return jspb.Message.setProto3StringField(this, 2, value);
};


/**
 * repeated float point = 3;
 * @return {!Array<number>}
 */
proto.PcdData.prototype.getPointList = function() {
  return /** @type {!Array<number>} */ (jspb.Message.getRepeatedFloatingPointField(this, 3));
};


/**
 * @param {!Array<number>} value
 * @return {!proto.PcdData} returns this
 */
proto.PcdData.prototype.setPointList = function(value) {
  return jspb.Message.setField(this, 3, value || []);
};


/**
 * @param {number} value
 * @param {number=} opt_index
 * @return {!proto.PcdData} returns this
 */
proto.PcdData.prototype.addPoint = function(value, opt_index) {
  return jspb.Message.addToRepeatedField(this, 3, value, opt_index);
};


/**
 * Clears the list making it empty but non-null.
 * @return {!proto.PcdData} returns this
 */
proto.PcdData.prototype.clearPointList = function() {
  return this.setPointList([]);
};


goog.object.extend(exports, proto);
```

{{< /details>}}

5.将`App.js`修改为如下代码

```js
import * as React from 'react';
import axios from 'axios';
import protobuf from 'protobufjs';
import pcdProto from './prc.proto';

const loadProtobuf = async () => {
  const root = await protobuf.load(pcdProto);
  return root.lookupType("PcdData");
};

const decodeProtobuf = async (buffer) => {
  const MyMessage = await loadProtobuf();
  return MyMessage.decode(new Uint8Array(buffer));
};

function App() {
  const [id, setId] = React.useState(0);
  const [name, setName] = React.useState("");
  const [points, setPoints] = React.useState([]);
  let url = `http://127.0.0.1:3100`;
  axios.get(url, { responseType: "arraybuffer" }).then(function (response) {
    decodeProtobuf(response.data).then(result => {
      setId(result.idx);
      setName(result.name);
      setPoints(result.point);
    })
  }).catch(function (error) {
    console.log(error);
  });

  // 只渲染20条数据
  const listPoints = points.slice(0, 21).map((point) =>
    <li>{point}</li>
  );

  return (
    <div style={{ marginLeft: 'auto', marginRight: 'auto', width: '80%', paddingTop: '30px' }}>
      点云文件id：{id}<br />
        点云文件名称：{name}<br />
        点云文件大小：{points.length}<br />
        点云数据信息：
      <ul>{listPoints}</ul>
    </div>
  );
}

export default App;
```

6.通过`npm start`启动该程序，然后浏览器中访问`http://127.0.0.1:3000`的结果类似如下，可以看出`Protocol Buffers`数据正常解码并展示

![点云数据展示](/blog_img/web/using-protobuf-in-react/point-cloud-render.png "点云数据展示")