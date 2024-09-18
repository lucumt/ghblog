---
title: "对多叉树进行序列化和反序列化"
date: 2020-06-26T21:06:21+08:00
lastmod: 2020-06-26T21:06:21+08:00
draft: true
keywords: ["二叉树","多叉树","序列化","反序列化"]
description: "基于代码的方式描述如何对多叉树进行序列化和反序列化"
tags: []
categories: ["算法"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
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

[**二叉树**](https://zh.wikipedia.org/zh-cn/%E4%BA%8C%E5%8F%89%E6%A0%91)是算法中面试中的高频题，本文介绍一种基于`二叉树`的`广度优先遍历`来实现将`多叉树`解析存储且能根据结果反向解析出`多叉树`的实现方案(即序列化和反序列化)。

<!--more-->

## 二叉树遍历

### 二叉树遍历方式

`二叉树`有如下所示的4种遍历方式[^1]

* **前序遍历**，首先访问根节点，然后遍历左子树，最后遍历右子树
* **中序遍历**，首先遍历左子树，然后访问根节点，最后遍历右子树
* **后序遍历**，首先遍历左子树，然后遍历右子树，最后访问根节点
* **广度遍历**[^2]，按照从左到右的顺序，逐层遍历各个节点

基于`多叉树`的特性(子节点多余2个，无法分辨具体的左右子节点)，只能实现`前序遍历`[^3]和`广度遍历`，本文出于算法实现和容易理解的角度采用`广度遍历`来实现`多叉树`的解析与恢复。

### 广度优先遍历图解

`二叉树`的`广度优先`遍历通常需要采用[**队列**](https://zh.wikipedia.org/wiki/%E9%98%9F%E5%88%97)这种数据结构来辅助进行，其遍历原理如下：

1. 对于不为空的结点，先把该结点加入到队列中

2. 从队中拿出结点，如果该结点的左右结点不为空，就分别把左右结点加入到队列中

3. 重复以上操作直到队列为空

![二叉树示意图](/blog_img/algorithm/serialization-and-deserialization-for-multiple-node-tree/binary-tree-structure.png "二叉树示意图")  

以上图所示的`二叉树`为例，其`广度优先`的实现代码如下:

```javascript
var Node = function(val){
	this.val = val;
	this.left  = null;
	this.right = null;
}

/**
			 A
		   /   \
		  B     C
		/  \  /  \
	   D   E  F   G
*/
function buildNode() {
	let node1 = new Node("A");
	let node2 = new Node("B");
	let node3 = new Node("C");
	let node4 = new Node("D");
	let node5 = new Node("E");
	let node6 = new Node("F");
	let node7 = new Node("G");

	node1.left = node2;
	node1.right = node3;

	node2.left = node4;
	node2.right = node5;

	node3.left = node6;
	node3.right = node7;

	return node1;
}

function traverseByBFS(root) {
	let queue = [root];
	let count = 1;
	while (queue.length > 0) {
		let node = queue.shift();
		document.write(node.val + " ");
		count--;
		if (!!node.left) {
			queue.push(node.left);
		}
		if (!!node.right) {
			queue.push(node.right);
		}
		if (count == 0) {
			document.write("<br/>");
			count = queue.length;
		}
	}
}

let root = buildNode();
traverseByBFS(root);
```

上述代码的执行结果如下：

```text
A
B C
D E F G
```

上述代码实现`广度遍历`的关键点如下：

1. 通过`队列`在遍历到当前节点时，将其左右子节点加入`队列`中，由于`队列`具有**先进先出**特性可确保只有同一层级的节点遍历完成后才会遍历下一层级
2. 通过定义变量count来记录每一层级的总数，每次输出一个节点值时，count值减1，当count值为0时，此时当前层级的节点已全都遍历完毕(此时`队列`中仍然有值，但存储的都是下一层级的数据)，输出换行符，同时将count值重新设置为`队列`的大小，开启新一轮循环
3. 循环结束条件为`队列`为空

## 多叉树序列化

### 多叉树的常规遍历

由于`广度遍历`不涉及到具体的左右子节点的区分，所以前述的遍历算法略加改进之后就可以适用于`多叉树`

### 在遍历的过程中存储位置关系

## 多叉树反序列化

[^1]:  通常的划分方式是将`前序遍历`、`中序遍历`、`后序遍历`归为`深度优先遍历`，主要采用递归方式实现
[^2]:  通常称作`广度优先遍历`，本文处于简化考虑统一称作`广度遍历`
[^3]: 实际上由于不能准确的区分`多叉树`的左右子节点，所以`前序遍历`在严格意义上来说也是无法实现的。

参考文档：

* https://www.cnblogs.com/three-fighter/p/15221956.html
* https://zhuanlan.zhihu.com/p/136758152
* https://www.cnblogs.com/Lanly/p/6305766.html

* https://blog.csdn.net/C20180602_csq/article/details/70738280
* https://blog.csdn.net/weixin_43314519/article/details/106981900
