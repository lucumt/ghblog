---
title: "将3个小int整数合并到1个long中从而缩小数据量"
date: 2022-09-08T10:04:59+08:00
lastmod: 2022-09-08T10:04:59+08:00
draft: false
keywords: ["bit","int","long"]
description: "简要介绍如何将3个小int整数合并到1个long中从而缩小数据量"
tags: ["bit"]
categories: ["位操作","Java编程"]
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
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true

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

近期在工作中遇到由于`HTTP`返回的内容较多导致系统响应延迟的问题，最终自己结合[gzip](https://en.wikipedia.org/wiki/Gzip)、[Protocol Buffers](https://developers.google.com/protocol-buffers)、[位运算](https://zh.wikipedia.org/wiki/%E4%BD%8D%E6%93%8D%E4%BD%9C)等将`HTTP`响应返回的数据从15M减少到1M从而解决系统无卡顿问题。其中对于`位运算`部分自己是结合业务实际，将3个小型`int`转化为1个`long`，将数据量减少三分之一，简单记录下其实现(以`Java`实现为例)。

<!--more-->

## 背景

某个项目模块要将一系列的坐标数据返回给前端页面，坐标数据由x、y、z这3个维度来定义且用`float`表示，它们的约束条件如下：

> $|x|$<150、$|y|$<50、$|z|$<50

示例文件如下：

```bash
123.45	34.56	23.78
111.35	-32.56	21.78
103.77	24.83	-13.78
#...共有超过10万行数据
99.83	21.35	43.39
```

最开始自己是采用自己是将它们逐行读取并以`String`的形式封装到`JSON`中返回给前端，此种原始的方式很明显会导致返回给前端的数据量巨大从而严重影响性能。

接下来自己从如下几个方面着手优化

1. 将返回的数据从`String`转化为`float`，减少整体响应的字节数
2. 采用`gzip`压缩，将响应数据的体积缩小到原始的三分之一
3. 采用`Protocol Buffers`在加快解析速度的同时进一步缩小请求响应的数据包体积

通过上述操作之后，响应速度从原来的4-5s缩短到1s之内，基本上达到优化目的，在后续的总结中发现请求响应慢的主要原因是由于响应头的体积过大造成的，于是继续寻找减少响应体积的办法。

## 分析

在`Java`中`int`和`float`都占用4个字节共32bit位，将上述示例文件中的坐标数据从`float`转化为`int`并不会造成响应数据的体积变化，但由于业务的特殊性限制了x、y、z这3个坐标的取值范围(有前述所述的范围限制)，尝试分析它们的占用的最大bit位数：

```java
x = 14999 = 0b100100100111101111 // 占用18bit位
y = 4999 = 0b1100001101001111 // 占用16bit位
z = 4999 = 0b1100001101001111 // 占用16bit位   
```

在不考虑负数的情况下，x、y、z累计占用的bit位数为50，而一个`long`类型占用8个字节共64bit位，理论分析是可以将3个`int`合并为1个`long`型数据。

![int和long占用的bit位数](/blog_img/bit/merge-three-int-number-into-a-long-number/bits-used-by-int-and-long.png "int和long占用的bit位数")  

### 存储设计

考虑到坐标数据可能有负数,可用1个bit位专门记录对应的坐标数是否为负数，此时加上正负数的标记也总共只占用53个bit位，没有超过单个`long`型数据的最大bit位数。

从便于编码的角度对x、y、z在`long`型中的存储做出如下划分:

> 每个`int`占用20个bit位，最前面一个bit位记录正负数(为正数时该bit位为0，为负数时该bit位为1)

![在long中存储int的实现方案](/blog_img/bit/merge-three-int-number-into-a-long-number/using-long-to-store-int-data.png "在long中存储int的实现方案")  

### 移位操作

前述理论分析可能，但在实际操作过程中会遇到如下2个问题：

1. 如何在对应的正负数标识bit位上写入和读取值
2. 如何给特定的坐标值准确写入对应的bit位且能准确读取

结合位运算本身特性以及实际业务需求，上述问题的解决方案如下：

1. 若特定坐标为负数，可将第20或40或60bit位设置为1

2. 为了准确记录各坐标数的值，需要将它们与一个类似`0b1111111111111111111`（十六进制表示为`0x7ffff`）的基准值进行`&`运算，由于要在单个`long`中记录下3个`int`的数值，此时就涉及到对坐标数据进行移位运算，如下图所示

   ![不同位置的坐标进行移位操作](/blog_img/bit/merge-three-int-number-into-a-long-number/int-data-bit-operate-to-integrate.png "不同位置的坐标进行移位操作")  

具体的分析如下：

* x为第3个存储，需要与`0b10000000000000000000`做`|`运算来记录正负数，需要与`0b01111111111111111111`做`&`来记录具体值
* y为第2个存储，需要与`0b1000000000000000000000000000000000000000`做`|`运算来记录正负数，需要与`0b0111111111111111111111111111111111111111`做`&`来记录具体值
* z为第1个存储，需要与`0b100000000000000000000000000000000000000000000000000000000000`做`|`运算来记录正负数，需要与`0b011111111111111111111111111111111111111111111111111111111111`做`&`来记录具体值

上述涉及到的6个基准数据用二进制表示太复杂，其精简表示如下：

* `0b10000000000000000000` => `1<<19`
* `0b1000000000000000000000000000000000000000` => `1L<<39`[^1]
* `0b100000000000000000000000000000000000000000000000000000000000`=> `1L<<59`
* `0b01111111111111111111` => `0x7ffff`
* `0b0111111111111111111111111111111111111111` => `0x7ffffL<<20`
* `0b011111111111111111111111111111111111111111111111111111111111`= >`0x7ffffL<<40`

若用`position`[^2]表示第几个坐标数，则基准数据可进一步精简如下：

* 正负数的记录位`1L << (position * 20 - 1)`
* 数据值的记录位`0x7ffffL << ((position - 1) * 20)`

## 实现

基于前述分析过程，坐标数据写入的代码如下，需要注意的是在写入过程中要传递一个类型为`long`的`target`[^3]变量，通过`target`变量来记录对应的坐标的具体数据和正负数标识。

```java
public static long bitWrite(int position, long original, long target) {
    // 记录是否为正负数的标识位
    long typeBase = 1L << (position * 20 - 1);
    // 记录具体的坐标数值
    long valueBase = 0x7ffffL << ((position - 1) * 20);
    
    // 若为负数，则需要将对应标识位记为1，同时将其变为正数
    if (original < 0) {
        target = target | typeBase;
        original = -original;
    }
    
    // 坐标数与数据值bit位做或操作记录下其值
    original = original << (position - 1) * 20;
    original = original & valueBase;
    
    // 将坐标数写入long型结果中
    target = target | original;
    return target;
}
```

坐标数据的读取过程则是写入操作的反向操作，需要将`&`变为`|`，将`|`变为`&`，同写入操作类似，需要传入之前写入的`target`变量，以便程序反向识别出相关数据

```java
public static long bitRead(int position, long target) {
    long value;
    long typeBase = 1L << (position * 20 - 1);
    
    //识别是否为负数
    boolean isNegative = (target & typeBase) == typeBase;

    // 读取具体的数值，此时读取的均为正数
    long valueBase = 0x7ffffL << ((position - 1) * 20);
    value = target & valueBase;
    value = value >> (position - 1) * 20;

    // 若为负数，则要进行恢复操作
    if (isNegative) {
        value = -value;
    }
    return value;
}
```

基于上述方法利用如下代码进行测试

```java
public static void testData() {
    int x = -14997;
    int y = 3349;
    int z = -2377;
    long target = 0;
    target = bitWrite(1, x, target);
    target = bitWrite(2, y, target);
    target = bitWrite(3, z, target);
    System.out.println(Long.toBinaryString(target));
    System.out.println(bitRead(1, target));
    System.out.println(bitRead(2, target));
    System.out.println(bitRead(3, target));
}
```

测试结果如下，程序输出结果符合预期

![测试结果](/blog_img/bit/merge-three-int-number-into-a-long-number/bits-merge-test-result.png "测试结果")  

完整代码参见[**SmallIntBitCompressTest.java**](https://github.com/lucumt/myrepository/blob/master/bit/SmallIntBitCompressTest.java)



[^1]: 由于`int`型最大的位数为32位，左移39位时超过其限制会编译出错，故需要使用`1L<<39`替代`1<<39`
[^2]: 与`Java`中的下标表示保持一致，`position`从0开始，故其值只能为0、1、2三者之一
[^3]: `target`变量的类型初始值需要设置为0