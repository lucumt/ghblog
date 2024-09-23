---
title: "Arrays.asList类型转换结果的坑"
date: 2019-07-22T10:47:12+08:00
lastmod: 2022-02-22T10:47:12+08:00
draft: false
keywords: ["Arrays.asList","类型转换","封装","原始类型"]
description: "记录下在java中使用Arrays.asList进行类型转换结果的坑以及解决方案"
tags: ["java"]
categories: ["java编程"]
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

在各种`Java`编程规范中都强调尽量不用[java.util.Arrays.asList()](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#asList-T...-)方法以避免调用此方法生成的`List`无法进行修改操作，个人近期在使用过程中发现该方法的另外一个坑，简单记录下。

<!--more-->

## 问题描述

在程序中需要检测某个元素是否在数组中，处于缩短代码篇幅的考虑，自己采用了`Arrays.asList`方法将其转化为`List`然后调用`contains`方法来判断：

```java
public static boolean checkContains(int ele) {
    int[] data = {11, 12, 13, 14};
    return Arrays.asList(data).contains(ele);
}
```

但实际执行结果和自己预期的不一致，理论上的结果应该为`true`，而实际返回的结果为`false`

![Arrays.asList输出结果不符合预期](/blog_img/java-core/arrays-as-list-convert-result-in-java/arrays-as-list-contains-result-not-as-expected.png "Arrays.asList输出结果不符合预期")  

## 问题分析

在Debug模式下查看`Arrays.asList(data)`返回的值，发现其返回的结果不是预期中的`List<Integer>`而是`List<int[]>`，从而导致调用`List.contains()`方法失效!

![Arrays.asList输出不符合预期的debug](/blog_img/java-core/arrays-as-list-convert-result-in-java/arrays-as-list-contains-variable-debug.png "Arrays.asList输出不符合预期的debug")  

在IDEA中将`Arrays.asList(data)`返回的结果单独赋值给一个变量，可发现其类型确实为`List<int[]>`，若将结果类型修改为`List<Integer>`则会报错，如下图所示：

![修改代码将转化结果抽取出来](/blog_img/java-core/arrays-as-list-convert-result-in-java/arrays-as-list-result-to-variable-compare.png "修改代码将转化结果抽取出来")  

在上图右边的报错信息中有如下说明信息:

> no instance(s) of type variable(s) exist so that int[] conforms to Integer inference variable T has incompatible bounds: equality constraints: Integer lower bounds: int[]

上述文字的核心内容为`Arrays.asList`需要获取`Object`类型，而`int`是原生类型，但`int[]`符合要求，因此`Arrays.asList()`方法会将原始的`int`数组视作一个`int[]`对象进行处理，从而导致此结果[^1]。

## 解决方案

找到问题原因后修改起来也很容易，按照通常的编程习惯，只需要把数组的定义从`int`修改为`Integer`即可

```java
public static boolean checkContains(int ele) {
    Integer[] data = {11, 12, 13, 14};
    List<Integer> dataList = Arrays.asList(data);
    return dataList.contains(ele);
}
```

执行结果符合预期

![Arrays.asList输出结果符合预期](/blog_img/java-core/arrays-as-list-convert-result-in-java/arrays-as-list-contains-result-as-expected.png "Arrays.asList输出结果符合预期")  

---

Update:

在`Stackoverflow`上找到一个类似问题[^2]，在其回答中提供了一些其它实现方案：

* `JDK8`实现

  ```java
  int[] ints = new int[] {1,2,3,4,5};
  List<Integer> list11 =Arrays.stream(ints).boxed().collect(Collectors.toList()); 
  ```

* `JDK16`实现

  ```java
  int[] ints = new int[] {1,2,3,4,5};
  Arrays.stream(ints).boxed().toList();
  ```

## asList不可变原因分析

查看`Arrays.asList`返回结果中的`List`，发现其返回的是内部自己实现的`ArrayList`，但该`ArrayList`没有重写`add`方法。

![Arrays.asList实现了自定义的ArrayList](/blog_img/java-core/arrays-as-list-convert-result-in-java/arrays-as-list-create-custom-arraylist.png "Arrays.asList实现了自定义的ArrayList")  

但[AbstractList](https://docs.oracle.com/javase/8/docs/api/java/util/AbstractList.html)默认没有实现`add()`方法而是抛出一个异常，从而导致无法通过调用`add()`方法添加元素。

![AbstractList默认不支持add操作](/blog_img/java-core/arrays-as-list-convert-result-in-java/abstract-list-add-method-throw-exception.png "AbstractList默认不支持add操作")  

[^1]: https://stackoverflow.com/questions/27522741/incompatible-types-inference-variable-t-has-incompatible-bounds
[^2]:https://stackoverflow.com/questions/2607289/converting-array-to-list-in-java 