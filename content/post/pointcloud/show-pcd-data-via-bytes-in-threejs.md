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

## 点云数据渲染

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

由于`PCDLoader`的封装太重，需要重新实现。

参考网络上的相关资料，修改后的代码如下，其通过`renderPcd()`函数接收点云数据集合并渲染，适用性更广。

```javascript
import axios from 'axios';
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { PcdData } from '../proto/pcd_pb.js';

let camera, scene, renderer;
let parent, width, height;

function initComponments() {
    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(width, height);
    parent.appendChild(renderer.domElement);

    scene = new THREE.Scene();

    camera = new THREE.PerspectiveCamera(12, width / height, 0.5, 50000);
    camera.position.z = 310;
    scene.add(camera);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.addEventListener('change', render); // use if there is no animation loop
    controls.target = new THREE.Vector3(0, 0, 1);
    controls.autoRotate = false;
    controls.dampingFactor = 0.25;

    // 控制缩放范围
    //controls.minDistance = 0.1;
    //controls.maxDistance = 100;

    //scene.add( new THREE.AxesHelper( 1 ) );

    window.addEventListener('resize', onWindowResize);
}

// 此处的data为点云数据集合
function renderPcd(data) {
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
    let positions = Float32Array.from(data.getPointList())

    let color = []
    for (let i = 0; i < data.getPointList().length; i += 3) {
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
    points.name = data.getName();

    // 图像缩放
    points.scale.set(1.2, 1.2, 1.2);
    scene.add(points);
    render();
}

function onWindowResize() {
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
    renderer.setSize(width, height);
    render();
}

function render() {
    renderer.render(scene, camera);
}
```

## 点云数据传输