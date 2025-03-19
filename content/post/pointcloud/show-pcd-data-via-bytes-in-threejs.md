---
title: "在three.js中加载字节形式的点云数据并利用Draco加快渲染速度"
date: 2025-03-11T10:13:14+08:00
lastmod: 2025-03-11T10:13:14+08:00
draft: true
keywords: ["点云","three.js","draco","快速渲染"]
description: "介绍如何在three.js加载字节形式的点云数据,并基于Draco的压缩与解压减少网络传输中的数据体积,加快渲染速度"
tags: ["pointcloud","draco"]
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
borderImage: true
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

基于[利用Draco对点云数据进行编码解码以实现高效网络传输](/post/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data)一文介绍个人项目中关于[**three.js**](https://threejs.org)和[**Draco**](https://github.com/google/draco)的使用实践。

<!--more-->

## threejs渲染

在 `three.js`的官方说明中对于对于点云数据的加载主要是通过[PCDLoader](https://threejs.org/docs/#examples/en/loaders/PCDLoader)，其官方的使用说明如下，十分简洁。

```javascript
// instantiate a loader
const loader = new PCDLoader();

// load a resource
loader.load(
    // resource URL
    'pointcloud.pcd',
    // called when the resource is loaded
    function(points) {
        scene.add(points);
    },
    // called when loading is in progress
    function(xhr) {
        console.log((xhr.loaded / xhr.total * 100) + '% loaded');
    },

    // called when loading has errors
    function(error) {
        console.log('An error happened');
    }
);
```

上述API是通过直接加载`pcd`点云文件的方式实现的，而我们常规的使用场景是`pcd`文件在服务器端，需要在客户端读取并渲染，涉及到字节流的传输，此时客户端不存在现成的`pcd`文件，需要接收字节形式的点云数据。

![基于网络传输点云文件](/blog_img/pointcloud/show-pcd-data-via-bytes-in-threejs/pcd-transmit-via-network.png "基于网络传输点云文件") 

上述基于`PCDLoader`实现的代码虽然很简洁，但其只能读取文件或URL数据流，使用范围受到一定程度的约束，个人希望在项目中能直接基于点云数据集合进行渲染。

参考网络上的相关资料，修改后的代码如下，其通过`renderPcd()`函数接收点云数据集合并渲染，适用性更广。

{{% codetabs %}}  

```javascript::渲染代码
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

export const initComponments = (width, height, parent) => {
    let renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(width, height);
    parent.appendChild(renderer.domElement);

    let scene = new THREE.Scene();

    let camera = new THREE.PerspectiveCamera(12, width / height, 0.5, 50000);
    camera.position.z = 310;
    scene.add(camera);

    let controls = new OrbitControls(camera, renderer.domElement);
    controls.addEventListener('change', () => renderer.render(scene, camera)); // use if there is no animation loop
    controls.target = new THREE.Vector3(0, 0, 1);
    controls.autoRotate = false;
    controls.dampingFactor = 0.25;

    // 控制缩放范围
    //controls.minDistance = 0.1;
    //controls.maxDistance = 100;

    //scene.add( new THREE.AxesHelper( 1 ) );

    window.addEventListener('resize', () => {
        camera.aspect = width / height;
        camera.updateProjectionMatrix();
        renderer.setSize(width, height);
        renderer.render(scene, camera);
    });
    return [renderer, camera, scene];
}

export const renderPcd = (name, data, renderer, camera, scene) => {
    // 移除旧点云数据
    for (let child of scene.children) {
        if (child.type === 'Points') {
            scene.remove(child);
            break;
        }
    }

    let geometry = new THREE.BufferGeometry();
    let material = new THREE.PointsMaterial({ size: 0.05, vertexColors: 2 });  //vertexColors: THREE.VertexColors
    let points = new THREE.Points(geometry, material);
    let positions = Float32Array.from(data);

    let color = []
    for (let i = 0; i < data.length; i += 3) {
        color[i] = 0.12;
        color[i + 1] = 0.565;
        color[i + 2] = 1;
    }
    let colors = new Float32Array(color)

    geometry.setAttribute("position", new THREE.BufferAttribute(positions, 3));
    geometry.setAttribute("color", new THREE.BufferAttribute(colors, 3));
    geometry.center();
    geometry.rotateX(Math.PI);

    // 沿y轴方向平移一定单位
    //points.translateY(10);
    //points.translateX(70);
    points.name = name;

    // 图像缩放
    points.scale.set(1.2, 1.2, 1.2);
    scene.add(points);
    renderer.render(scene, camera);
}
```

```javascript::调用代码
import { PcdData } from '../proto/pcd_pb';
import { request } from '../tools/axios_tools';
import { initComponments, renderPcd } from '../tools/play_tools';

let renderer, camera, scene;
export const clearPcdData = () => {
    renderer = null;
    camera = null;
    scene = null;
}

export const playPcd = (pId, pHeight, data, playingRef, updateState) => {
    // 进行一些必要的初始化操作
    let parent = document.getElementById(`${pId}`);
    let width = parent.offsetWidth;

    let total = data.total;
    if (total === 0) {
        return;
    }
    if (!scene) {
        let [sRenderer, sCamera, sScene] = initComponments(width, pHeight, parent);
        renderer = sRenderer;
        camera = sCamera;
        scene = sScene;
    }
    loadPlayPcd(data, playingRef, updateState, renderer, camera, scene);
};

let count = 0;
export const loadPlayPcd = (data, playingRef, updateState, renderer, camera, scene) => {
    let file = data.file[count];
    let total = data.total;
    count++;
    let url = `main/loadPcdBinary?pcd=${file}`;
    request.get(url, { responseType: "arraybuffer" }).then(function (response) {
        if (!playingRef.current) {
            return;
        }
        let result = PcdData.deserializeBinary(response.data);
        renderPcd(result.getName(), result.getPointList(), renderer, camera, scene);
        if (count < total) {
            let percent = (count / total * 100).toFixed(2);
            updateState({ 'progress': percent, processCount: count });
            loadPlayPcd(data, playingRef, updateState, renderer, camera, scene)
        } else {
            updateState({ 'progress': 0, processCount: 0 });
            count = 0;
        }
    }).catch(function (error) {
        console.log(error);
    });
};
```
{{% /codetabs %}}  

## 服务端实现

服务端采用[Express](https://expressjs.com)实现，其主要功能如下：

1. 以`JSON`格式显示`pcd`、`ply`、`drc`等点云文件的整体信息
2. 将`pcd`文件转化为`ply`或`drc`格式的点云文件，相关操作均基于`JavaScript`实现
3. 以`Protocol Buffers`编码的形式提供`pcd`、`ply`、`drc`格式点云文件的下载，供客户端下载并渲染

相关代码位于[draco-point-cloud-server](https://github.com/fox-world/draco-point-cloud-server)，可将其下载到本地直接运行，也可基于下述相关指令构建`Docker`镜像并运行，或采用个人已经提前构建好的镜像[draco_server](https://hub.docker.com/repository/docker/lucumt/draco_server)

{{% codetabs %}}  

```bash::操作指令
# build image
docker build -t lucumt/draco_server:1.0 .

# run docker
docker run -d -p 8000:8000 --name draco_server lucumt/draco_server:1.0

# stop docker
docker stop draco_server && docker rm draco_server
```

```dockerfile::Dockerfile
FROM node:22.14.0-alpine

RUN mkdir /draco

COPY draco-point-cloud-server.zip /draco/draco-point-cloud-server.zip
WORKDIR /draco

RUN unzip draco-point-cloud-server.zip && rm draco-point-cloud-server.zip
RUN mv draco-point-cloud-server server

WORKDIR /draco/server
RUN npm i

ENV PORT=8000
CMD ["node", "./bin/www"]
```

{{% /codetabs %}}  

启动该服务后在浏览器中打开，其主页面显示如下

![Draco服务端运行界面](/blog_img/pointcloud/show-pcd-data-via-bytes-in-threejs/draco-server-home-page.png "Draco服务端运行界面") 

点击相关链接即可进行相关操作，而基于`JavaScript`通过`Draco`将`pcd`转化为`drc`文件十分耗时，故推荐采用脚本进行批量转化以加快转化时间，具体请参见[脚本转换](/post/pointcloud/using-draco-to-encode-decode-and-transport-pointcloud-data/#脚本转换)。



为了方便使用，个人提前在该项目中已经准备好了若干`pcd`和`drc`格式的点云数据，通过源码或镜像启动服务端后可直接让客户端下载相关数据并进行渲染。

![Draco测试文件列表](/blog_img/pointcloud/show-pcd-data-via-bytes-in-threejs/draco-server-file-list.png "Draco测试文件列表") 

## 客户端实现

客户端基于[React](https://react.dev/)实现，其主要功能如下：

1. 获取服务端`pcd`和`drc`文件的总数信息，并在页面展示
2. 下载`pcd`或`drc`文件，基于`Protocol Buffers`进行解码，并对`drc`文件通过`Draco`进行进一步的解码
3. 针对`pcd`和`drc`文件，分别基于`threejs`进行点云渲染，可进行开始播放、暂停播放、切换播放源等操作

相关的代码位于[draco-point-cloud-client](https://github.com/fox-world/draco-point-cloud-client)，可将其下载到本地自行编译部署，也可基于下述相关指令构建`Docker`镜像并运行，或采用个人已经提前构建好的镜像[draco_client](https://hub.docker.com/repository/docker/lucumt/draco_client)

{{% codetabs %}}  

```bash::操作指令
# build image
docker build -t lucumt/draco_client:1.0 .

# run docker
docker run -d -p 3000:3000 --name draco_client lucumt/draco_client:1.0

# stop docker
docker stop draco_client && docker rm draco_client
```

```dockerfile::Dockerfile
FROM nginx

COPY dist /usr/share/nginx/html
COPY draco.conf /etc/nginx/conf.d/

EXPOSE 3000
```

```nginx::Nginx配置文件
server {
    listen       3000;
    server_name  draco-server;

    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST,PUT,DELETE, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';


    location / {
      root   /usr/share/nginx/html;
      index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

{{% /codetabs %}}  

启动该服务后在浏览器中打开，其主页面显示如下

![Draco客户端界面](/blog_img/pointcloud/show-pcd-data-via-bytes-in-threejs/draco-client-default-page.png "Draco客户端界面") 

点击图中的播放按钮，会加载点云文件并基于`threejs`进行渲染展示，或者通过顶部的tab菜单切换不同的数据格式播放

![Draco客户端点云渲染](/blog_img/pointcloud/show-pcd-data-via-bytes-in-threejs/draco-client-render-page.png "Draco客户端点云渲染") 

## 错误修复

## 效果展示对比