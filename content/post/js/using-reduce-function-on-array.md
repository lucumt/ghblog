---
title: "JavaScript学习-reduce函数"
date: 2022-09-26T09:39:47+08:00
lastmod: 2022-09-26T09:39:47+08:00
draft: false
keywords: []
description: "简要记录自己对于JavaScript中reduce函数的学习，供学习与总结"
tags: ["javascript"]
categories: ["web编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
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

简要记录下自己对于`JavaScript`中[**reduce()**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)函数的学习小结。

<!--more-->

原始问题来源于[**How to sum an array for each ID and create new array in react**](https://stackoverflow.com/questions/74334249/how-to-sum-an-array-for-each-id-and-create-new-array-in-react/)，问题如下：

有一个数组

```javascript
const data = [
  {"id": "One", "number": 100}, 
  {"id": "One", "number": 150}, 
  {"id": "One", "number": 200}, 
  {"id": "Two", "number": 50}, 
  {"id": "Two", "number": 100}, 
  {"id": "Three", "number": 10}, 
  {"id": "Three", "number": 90}
];
```

提问者希望基于`id`对`number`进行累加，并输出一个新的数组：

```javascript
[
  {"id": "One", "number": 450},
  {"id": "Two", "number": 150}, 
  {"id": "Three", "number": 100}, 
]
```

由于自己当时正在`Stackoverflow`上刷`reputation`，所以看到这个问题后，毫不犹豫的写下了自己的答案

```javascript
let result = data.reduce((a, v) => {
  let obj = a.find(i => i.id == v.id);
  if (obj) {
      obj.number += v.number;
  } else {
      a.push({...v});
  }
  return a;
}, [])
console.log(result);
console.log("-------------------------------")
```

虽然我的手速比较快，但是最后被原提问者选中的答案如下

```javascript
let result = data.reduce((acc, {id, number}) => ({...acc, [id]: {id, number: acc[id] ? acc[id].number + number: number}}), {});
console.log(Object.values(result));
```

该写法通过给`reduce`函数的输出结果设置为`{}`对象的方式避免了需要判断是是否为空，最后通过[**Object.values()**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Object/values)将结果转化为数组，相对于我之前的多行代码，此种写法只需要2行即可实现目的，更简洁更优化，学习了！