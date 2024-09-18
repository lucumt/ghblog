---
title: "不通过第三方变量来交换两个变量的值"
date: 2016-08-12T11:03:09+08:00
lastmod: 2016-08-12T11:03:09+08:00
draft: false
keywords: ["交换变量","位运算"]
description: ""
tags: ["bit"]
categories: ["算法","Java编程","位运算"]
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

变量交换是编程中经常使用的功能，本文记录几种不通过不添加第三方变量来交换两个变量的实现方式。

<!--more-->

交换变量通常采用类似如下代码：

```java
int a = 3, b = 4, temp;
temp = a;
a = b;
b = temp;
System.out.println(a);
System.out.println(b);
```

上述代码实现和理解起来都很容易，除此之外还有其它的实现方式，本文以非0的int变量为例，简单记录下自己了解的相关实现。

## 加减法实现

```java
int a = 3, b = 4;
a = a + b;
b = a - b;
a = a - b;
System.out.println(a);
System.out.println(b);
```

上述代码可以进一步精简如下:

```java
int a = 3, b = 4;
a = a + b - (b = a);
System.out.println(a);
System.out.println(b);
```

## 乘除法实现

```java
int a = 3, b = 4;
a = a * b;
b = a / b;
a = a / b;
System.out.println(a);
System.out.println(b);
```

## 位运算实现

```java
int a = 3, b = 4;
a = a ^ b;
b = a ^ b;
a = a ^ b;
System.out.println(a);
System.out.println(b);
```



