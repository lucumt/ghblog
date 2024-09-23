---
title: "ClassNotFoundException与NoClassDefFoundError对比"
date: 2019-01-02T14:18:46+08:00
lastmod: 2019-01-02T14:18:46+08:00
draft: false
keywords: ["ClassNotFoundException","NoClassDefFoundErro"]
description: "简要对比说明在java开发中经常遇到的ClassNotFoundException与NoClassDefFoundError对比"
tags: ["Java"]
categories: ["Java编程"]
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

[**ClassNotFoundException**](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassNotFoundException.html)与[**NoClassDefFoundError**](https://docs.oracle.com/javase/8/docs/api/java/lang/NoClassDefFoundError.html)是`Java`开发中经常会遇到的异常与错误，本文基于个人工作中遇到的场景以及网上的资料，简要总结它们的差异、出现场景以及规避方案。

<!--more-->

## ClassNotFoundException

### 分析

![ClassNotFoundException类结构](/blog_img/java-core/difference-between-class-not-found-exception-and-no-class-def-found-error/class-not-found-exception-structure.png "ClassNotFoundException类结构") 

上图为`ClassNotFoundException`的类继承结构，从图中可看出它继承自[Exception](https://docs.oracle.com/javase/8/docs/api/java/lang/Exception.html)且没有继承自[RuntimeException](https://docs.oracle.com/javase/8/docs/api/java/lang/RuntimeException.html),基于`Java`语言规范它属于`Checked Exception`[^1]，实际编程时必须在代码中利用`try-catch`显示的捕获处理或者利用`throws`将其抛出到上一层调用者处理，否则会导致编译器报错。

### 产生原因

在`ClassNotFoundException`的[官方API](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassNotFoundException.html)中对于其产生的原因有如下描述:

> Thrown when an application tries to load in a class through its string name using:
>
> - The `forName` method in class `Class`.
> - The `findSystemClass` method in class `ClassLoader` .
> - The `loadClass` method in class `ClassLoader`.
>
> but no definition for the class with the specified name could be found.

基于上述描述可知`ClassNotFoundException`产生的根本原因如下：

> **在应用程序类中通过对应类的字符串名称去加载该类时找不到类的定义文件，即会抛出此异常**

上述描述实际上可分为2类:

1. 在`Class`类中通过`forName`方法去加载类时找不到类定义文件，由于该方式可在普通的`Java`类中使用故较为常见(如加载数据库驱动)
2. 在`ClassLoader`类中通过`findSystemClass`或`loadClass`去加载类时找不到类定义文件，由于是在类加载器中使用，一般的业务涉及较少故不常见

### 问题复现

* 基于`Class`类的`forName`模拟复现:

  ```java
  public class TestClassForName {
  
      public static void main(String[] args) {
          testForName();
      }
  
      private static void testForName() {
          try {
              Class.forName("com.jdbc.mysql.Driver");
              System.out.println("output info after Class.forName invoke inside try-catch");
          } catch (ClassNotFoundException e) {
              e.printStackTrace();
          }
          System.out.println("output info after Class.forName invoke outside try-catch");
      }
  }
  ```

  输出结果如下，可见`try-catch`之后的代码块还能正常执行

  ![利用Class中的forName模拟](/blog_img/java-core/difference-between-class-not-found-exception-and-no-class-def-found-error/class-not-found-exception-produce-result-1.png "利用Class中的forName模拟")

* 基于`ClassLoader`中的`loadClass`复现：

  ```JAVA
  public class TestClassLoader {
  
      public static void main(String[] args) {
          testLoadClass();
      }
  
      private static void testLoadClass() {
          try {
              ClassLoader.getSystemClassLoader().loadClass("com.jdbc.mysql.Driver");
              System.out.println("output info after ClassLoader.loadClass invoke inside try-catch");
          } catch (ClassNotFoundException e) {
              e.printStackTrace();
          }
          System.out.println("output info after ClassLoader.loadClass invoke outside try-catch");
      }
  }
  ```

  输出结果如下

  ![利用ClassLoader中的loadClass模拟](/blog_img/java-core/difference-between-class-not-found-exception-and-no-class-def-found-error/class-not-found-exception-produce-result-2.png "利用ClassLoader中的loadClass模拟")

从上述结果可以看出`ClassNotFoundException`虽然会导致`try-catch`代码块里面位于异常发生点之后的代码无法执行，但是位于`try-catch`代码块外面的代码程序执行不受影响(这其实是`Checked Exception`自身的特性决定的)。

## NoClassDefFoundError

### 分析

![NoClassDefFoundError类结构](/blog_img/java-core/difference-between-class-not-found-exception-and-no-class-def-found-error/no-class-def-found-error-structure.png "NoClassDefFoundError类结构") 

上图为`NoClassDefFoundError`的类继承结构，从图中可看出它继承自[Error](https://docs.oracle.com/javase/8/docs/api/java/lang/Error.html),，在`Java`语言关于`Exception`的[官方API](https://docs.oracle.com/javase/specs/jls/se7/html/jls-11.html)中有如下描述：

> The *unchecked exception classes* are the run-time exception classes and the error classes.

故`NoClassDefFoundError`属于`Unchecked Exception`，而对于这类异常不需要在程序中显示的通过`try-catch`捕获，需要从代码和工程层面进行处理。

### 产生原因

在`NoClassDefFoundError`的[官方API](https://docs.oracle.com/javase/8/docs/api/java/lang/NoClassDefFoundError.html)中有如下描述：

> Thrown if the Java Virtual Machine or a `ClassLoader` instance tries to load in the definition of a class (as part of a normal method call or as part of creating a new instance using the `new` expression) and no definition of the class could be found.
>
> The searched-for class definition existed when the currently executing class was compiled, but the definition can no longer be found.

基于上述描述可知`NoClassDefFoundError`产生的根本原因如下：

> 程序在编译时该类存在，在调用过程中JVM虚拟机加载该类时找不到该类的Class文件

由于`Java`类加载过程有如下图所示的5个阶段，在其中每个阶段都可能会出现问题，结合实际开发场景，最典型的两类场景如下：

1. 类加载阶段出错。如在编译时类存在，但执行时不存在，一个典型的场景是 **jar包版本不匹配！** 这也是我们开发过程中最容易遇到的场景
2. 类初始化阶段出错。此种场景下编译也能通过，但由于某种原因导致初始化失败，程序仍然无法调用类。

![Java类初始化阶段](/blog_img/java-core/difference-between-class-not-found-exception-and-no-class-def-found-error/java-class-init-phase.jpg "Java类初始化阶段") 

与之相似的错误为[**NoSuchMethodError**](https://docs.oracle.com/javase/8/docs/api/java/lang/NoSuchMethodError.html)，其产生原因为程序执行时类存在，但是当前程序调用的方法不存在。

### 问题复现

* 类加载阶段出错，在编译后jar文件中删除要调用的class类即可实现：

  要编译为jar文件的代码

  ```java
  /* 类结构
  +---src
  |   +---main
  |   |   \---java
  |   |       \---com
  |   |           \---lucumt
  |   |               \---api
  |   |                       InnerApi.java
  |   |                       OuterApi.java
  */
  
  public class OuterApi {
  
      public static void invoke() {
          InnerApi.hello();
      }
  }
  
  public class InnerApi {
  
      public static void hello() {
          System.out.println("hello");
      }
  }
  ```

  主执行类

  ```java
  import com.lucumt.api.OuterApi;
  
  public class TestNoClassDefFoundError1 {
  
      public static void main(String[] args) {
          try {
              OuterApi.invoke();
          } catch (NoClassDefFoundError e) {
              e.printStackTrace();
          }
          System.out.println("OutApi invoke");
      }
  }
  ```

  将上述工程编译打包为jar文件，然后在jar文件中去掉InnerApi，将其引入包含执行类的测试工程，此时其结构如下：

  ![在jar文件中删除类](/blog_img/java-core/difference-between-class-not-found-exception-and-no-class-def-found-error/no-class-def-found-error-reproduce-1.png "在jar文件中删除类") 

  输出结果如下，可以看出虽然抛出了异常，但在`try-catch`代码块之后的代码还是能够执行 

  ![类不存在导致错误](/blog_img/java-core/difference-between-class-not-found-exception-and-no-class-def-found-error/no-class-def-found-error-result-1.png "类不存在导致错误") 

* 类初始化阶段出错，通过某种方式强制初始化出错：

  ```java
  
  public class TestNoClassDefFoundError2 {
  
      public static void main(String[] args) {
          try {
              Demo.hello();
          }catch(NoClassDefFoundError e){
              e.printStackTrace();
          }
          System.out.println("invoke Demo.hello()");
      }
  }
  
  class Demo {
      private static int val = 1 / 0;
  
    public static void hello() {
          System.out.println("hello");
    }
  }
  ```
  
  输出结果如下，不同于前一个测试，此处的`try-catch`之后的代码没有机会执行
  
  ![类初始化失败导致错误](/blog_img/java-core/difference-between-class-not-found-exception-and-no-class-def-found-error/no-class-def-found-error-result-2.png "类初始化失败导致错误") 

从上面2个实验中可发现虽然都对`NoClassDefFoundError`进行了`try-catch`捕获，但第一个能继续执行，而第二个却直接终止。造成这种差异的主要原因是第一个测试中的`Demo`没能完成初始化，导致整个程序加载失败(实际上程序提示的错误为 `ExceptionInInitializerError`)，而第二个程序是在执行中才遇到类缺失[^2]。

基于`Unchecked Exception`我们通常不需要在代码中显示的利用`try-catch`捕获，更多的是修改代码本身和调整jar文件版本。

## 总结

|                | **ClassNotFoundException**                                   | NoClassDefFoundError                                         |
| :------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 继承关系       | `java.lang.Exception`                                        | `java.lang.Error`                                            |
| 类型           | 异常                                                         | 错误                                                         |
| 程序受影响程度 | 添加`try-catch`后可继续执行                                  | 基于产生于类加载的不同阶段，可能直接种植，也可能在`try-catch`之后继续执行 |
| 可能的场景     | 1.要加载的类不存在<br>2.类名编写错误                         | 1.jar包缺失或jar包版本不匹配<br>2.类初始化失败               |
| 处理方法       | 1. 修改代码，添加缺失的jar文件<br>2. 代码中添加`try-catch`捕获异常 | 修改代码业务逻辑                                             |

参考:

* https://segmentfault.com/a/1190000021292121

* https://stackoverflow.com/questions/28322833/classnotfoundexception-vs-noclassdeffounderror

[^1]: https://docs.oracle.com/javase/specs/jls/se7/html/jls-11.html
[^2]: 实际上应该有更科学更理论的解释，只不过自己目前找不到相关资料，暂时这么解释