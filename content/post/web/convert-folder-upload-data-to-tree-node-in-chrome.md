---
title: "在Chrome中将上传的文件夹数据转化为按树结构展示"
date: 2022-02-20T20:38:18+08:00
lastmod: 2022-08-20T20:38:18+08:00
draft: false
keywords: ["文件夹上传","树结构"]
description: "简要叙述如何在Chrome中将上传的文件夹数据转化为按树结构展示"
tags: ["html5","JavaScript"]
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

项目中有个模块支持按文件夹选中批量上传，用户希望在真正上传之前能够在浏览器中实时预览选中的文件夹层级结构，本文基于`Chrome`浏览器为例，简要说明这一实现。

<!--more-->

若要`Chrome`浏览器中支持批量上传文件夹，需要在`input`控件中添加`webkitdirectory`属性，如下图所示

```html
<body>
  上传文件夹 <input type="file" webkitdirectory onchange="getfolder(event)">
</body>
```

以一个典型的`Maven`功能做为测试目录，其文件结构如下，用户希望选中SpringTest文件夹后在前端上传之前就能够在网页上展示出类似的结构

![项目结构](/blog_img/web/convert-folder-upload-data-to-tree-node-in-chrome/maven-project-structure.png "项目结构") 

当基于前述的`HTML`代码选中上面的工程目录上传时，`Chrome`浏览器会提示如下警告框，此警告框是浏览器出于安全原因提示[^1]，可不用理会。

![批量上传时的警告](/blog_img/web/convert-folder-upload-data-to-tree-node-in-chrome/chrome-batch-upload-alert.png "批量上传时的警告") 

## 结构分析

在`getfolder`方法中分析`event`变量时，可发现`event.target.files`属性下包含我们所上传的全部文件，将其打印输出的结果如下：

![批量上传的文件列表](/blog_img/web/convert-folder-upload-data-to-tree-node-in-chrome/event-target-file-list.png "批量上传的文件列表") 

进一步观察可发现其中的`webkitRelativePath`中包含文件的完整路径，类似`SpringTest/target/classes/com/lucumt/bean/User.class`，这个路径虽然包含了上传文件的完整路径，但其为一个字符串，无法直接用于展示层级结构，进一步分析`event`变量的其它属性也没有找到有用的信息，看来只能从`webkitRelativePath`着手更改。

## 修改思路

为了展示层级结构，首先需要定义一个如下所示的`Node`结构，其中`path`用于存储对应的文件名(不含父路径)，`children`在为文件夹时存储子文件(文件夹)的信息：

```javascript
function Node(path){
    this.path = path;
    this.children = [];
}
```

最终的结果都是基于`Node`进行组合处理。

### 逐层处理

![逐层处理](/blog_img/web/convert-folder-upload-data-to-tree-node-in-chrome/deal-path-by-layer.png "逐层处理") 

如上图所示，由于通过`JavaScript`获取的都是节点的完整路径，从简化实现的角度考虑，系统按照文件夹层级**自底向上**逐级解析，解析到根目录时则停止。在此过程中，获取到的文件路径不断缩小，直至处理到根节点，以`SpringTest/target/test-classes/com/lucumt/TestGetBean.class`为例，其解析过程说明如下：

1. 初始路径为`SpringTest/target/test-classes/com/lucumt/TestGetBean.class`
2. 将前述路径拆分为`TestGetBean.class`(为其创建节点node1)和`SpringTest/target/test-classes/com/lucumt`(为其创建节点node2)，node2的子节点包含node1
3. 将node2的路径拆分为`lucumt`(更新node2节点的路径)和`SpringTest/target/test-classes/com`(为其创建node3节点)，node3节点的子节点包含node2
4. 将node3节点的路径拆分为`com`(更新node3节点的路径)和`SpringTest/target/test-classes`(为其创建节点node4)
5. 将node4节点的路径拆分为`test-classes`(更新node4节点的路径)和`SpringTest/target`(为其创建节点node5)
6. 将node5节点的路径拆分为`target`(更新node5节点路径)和`SpringTest`(为其创建节点node6)
7. 节点node6已经是根节点，整个过程完毕，至此从node6基于`children`属性可一直往下招到node1文件节点。

### 处理重复

基于上述实现方案时有一个问题待解决，如`SpringTest/target/test-classes/com/lucumt/TestGetBean.class`和`SpringTest/target/classes/com/lucumt/TestApplication.class`在拆分到第2层级时，都会识别到`lucumt`文件夹，若都去创建该文件夹节点数据则会导致重复。

![路径重复](/blog_img/web/convert-folder-upload-data-to-tree-node-in-chrome/node-path-duplicate.png "路径重复") 

回顾前面的逐层处理实现方案可知，当处理`SpringTest/target/classes/com/lucumt/TestApplication.class`时，`SpringTest/target/test-classes/com/lucumt/TestGetBean.class`已经被处理完比，在这一过程中会给我们创建好`lucumt`目录，在接下来处理`SpringTest/target/test-classes/com/lucumt/TestGetBean.class`时我们只需要**找到`lucumt`对应的父节点，然后检查父节点下有没有该目录即可。**

基于上述分析，要在逐级遍历的过程中添加如下处理：

1. 遍历到当前文件或文件夹节点时，需要查找其父节点，若父节点不存在则创建，同时将当前节点加入到父节点的子节点集合中去
2. 若当前节点的父节点存在，则需要获取其所有的子节点，并与当前节点进行避免，判断当前节点是否存在，若存在则不需要重复创建

为了记录当前节点的父节点，可在`JavaScript`中采用`Map`数据结构，其中key为节点的路径，value为Node节点自身。

## 代码实现

```javascript
function getfolder(event) {
    let files = event.target.files;
    let map = {};
    var rootPath = files[0].webkitRelativePath.split('/')[0];
    // 循环遍历，检测所有的文件路径
    for (var i in files) {
        var node = files[i];
        var path = node.webkitRelativePath;
        if (!path) {
            continue;
        }
        while (true) {
            // 处理到最顶层文件夹时，就不用往上继续处理
            if (!path || path.indexOf("/") == -1) {
                break;
            }
            var index = path.lastIndexOf("/");
            var parentPath = path.substring(0, index);
            var file = path.substring(index + 1);
            var node;
            if (!map[path]) {
                node = new Node(file);
                map[path] = node;
            } else {
                node = map[path];
            }

            // 父节点没有则创建
            if (!map[parentPath]) {
                var parentFile = parentPath.substring(parentPath.lastIndexOf("/") + 1)
                var pNode = new Node(parentFile);

                // 动态的更新父节点的子节点，为后续检查重复节点做准备
                pNode.children.push(node);

                map[parentPath] = pNode;
            } else {
                var pNode = map[parentPath];
                var children = pNode.children;
                var addNode = true;

                // 通过检查父节点下的子节点来避免重复创建
                for (var k in children) {
                    if (children[k].path == file) {
                        addNode = false;
                        break;
                    }
                }
                if (addNode) {
                    pNode.children.push(node);
                }
            }
            // 逐级缩短父节点的路径，直到回到最顶层
            path = parentPath;
        }
    }
    console.log(map[rootPath]);

}
```

执行上述代码后在`Chrome`控制台的输出如下，可见`root`中已能够正确的输出层级结构，进行到这一步后续在页面上展示就很容易了，此处不再叙述。

![树结构展示](/blog_img/web/convert-folder-upload-data-to-tree-node-in-chrome/tree-structure-display.png "树结构展示") 

### Java版本实现

```java
public class TestFilePathConvert {

    public static void main(String[] args) {
        String[] paths = {
                "a/b1/c1/d1.txt",
                "a/b1/c1/d2.txt",
                "a/b1/c1/d2/e1/f.txt",
                "a/b2/c2/d1",
                "a/b3/c1",
                "a/b4/c1/d1/e1/f1",
                "a/b4/c1/d1/e1/f2/g1.png",
                "a/b5/c1"
        };
        convertPathToTreeNode(paths);
    }

    public static void convertPathToTreeNode(String[] paths) {


        Map<String, Node> map = new HashMap<>();
        for (String path : paths) {
            while (true) {
                Node node, pNode;
                String nodeName;
                if (map.containsKey(path)) {
                    node = map.get(path);
                    nodeName = node.getName();
                } else {
                    int index = path.lastIndexOf("/");
                    nodeName = path.substring(index + 1);
                    node = new Node(nodeName);
                    map.put(path, node);
                }

                String parentPath = StringUtils.substringBeforeLast(path, "/");
                if (path.equals(parentPath)) {
                    break;
                }

                // 处理父节点
                if (map.containsKey(parentPath)) {
                    pNode = map.get(parentPath);
                } else {
                    int index = parentPath.lastIndexOf("/");
                    String pName = parentPath.substring(index + 1);
                    pNode = new Node(pName);
                    map.put(parentPath, pNode);
                }
                //检查当前节点是否存在
                boolean add = true;
                for (Node n : pNode.getChildren()) {
                    if (nodeName.equals(n.getName())) {
                        add = false;
                        break;
                    }
                }
                // 避免当前节点的重复添加
                if (add) {
                    pNode.getChildren().add(node);
                }
                path = parentPath;
            }
        }
        String root = StringUtils.substringBefore(paths[0], "/");
        Node rootNode = map.get(root);
        Gson gson = new Gson();
        System.out.println(gson.toJson(rootNode));
    }

    static class Node {
        private String name;

        private List<Node> children = new ArrayList<>();

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public List<Node> getChildren() {
            return children;
        }

        public void setChildren(List<Node> children) {
            this.children = children;
        }

        public Node(String name) {
            this.name = name;
        }


        @Override
        public String toString() {
            return "Node{" +
                    "name='" + name + '\'' +
                    ", children=" + children.size() +
                    '}';
        }
    }
}
```





[^1]: https://stackoverflow.com/questions/50225019/how-to-remove-warning-message-in-chrome-when-uploading-a-directory