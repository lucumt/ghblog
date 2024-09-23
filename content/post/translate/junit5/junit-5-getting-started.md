---
title: "[译]JUnit 5 入门 - 编写第一段测试代码"
date: 2023-05-01T09:51:22+08:00
lastmod: 2023-05-01T09:51:22+08:00
draft: false
keywords: ["junit5"]
description: "JUnit5翻译专题，简要介绍如何开始编写基于JUnit5的测试程序"
tags: ["junit5","java","junit"]
categories: ["翻译","JUnit5翻译"]
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
centerImage: true
borderImage: false

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

本文翻译自[**Getting Started with JUnit 5: Writing Your First Test**](https://www.arhohuttunen.com/junit-5-getting-started)。

<!--more-->

![Get Started JUnit 5](/blog_img/translate/junit5/junit-5-getting-started/get-started-junit-5.webp "Get Started JUnit 5") 

在本文中，我们将学习如何编写和运行简单的`JUnit5`单元测试、如何设置前提条件、与我们想测试的对象进行交互，以及验证程序是否按照预期执行。



本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 设置

附带的源码示例已经包含基于`Maven`和`Gradle`的配置，但为了方便起见，同时也提供了分步教程指南。

{{% admonition note "补充阅读" false %}}

* [**JUnit5 Maven示例**](https://www.arhohuttunen.com/junit-5-maven-example/)
* [**JUnit5 Gradle示例**](https://www.arhohuttunen.com/junit-5-gradle-example/)

{{% /admonition %}}

## 编写第一个测试

当我们测试某段代码时，我们希望确保其能够按照预期来运行。在编写自动化测试时基于下述步骤：

1. 首先，对我们要测试的代码**设置前提条件**
2. 接下来，对代码进行**交互测试**
3. 最后，**检查执行结果**是否符合预期

这被称为**Arrange**,**Act**,**Assert**模式。

接下来看一下我们想要测试的代码：

```java
public class Calculator {
    public int add(int first, int second) {
        return first + second;
    }
}
```

这是一个非常简单的计算器类，主要用于两数相加，在测试此段代码时，我们希望计算结果是正确的。

我们可基于**Arrange**,**Act**,**Assert**模式按下述步骤实现：

1. 首先构造一个`Calculator`类的实例
2. 然后传递调用`add()`方法并将结果存储起来
3. 最后，通过调用`assertEquals()`来比较实际结果和预期结果

```java
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

class CalculatorTest {
    @Test
    void addNumbers() {
        // Arrange
        Calculator calculator = new Calculator(); <strong>x</strong>
        
        // Act
        int sum = calculator.add(1, 2);
        
        // Asset
        Assertions.assertEquals(3, sum);
    }
}
```

测试类可以用任何名称，但在本文中我们将测试代码放入`CalculatorTest`类中，我们将测试类简单叫做`addNumbers()`以便能够描述我们正在尝试做的事情。



为了能够执行测试，我们需要给测试方法加上`@Test`注解，通过这种方式测试执行器能将该方法识别为一个测试方法。

## 运行测试

我们可以有多种方式运行测试代码，当使用IDE工具时可以从IDE工具中直接运行，我们也可以使用像`Maven`或`Gradle`这样的构建工具在命令行中执行测试。

当软件(代码)按照预期执行时，测试结果为`pass`,当出现预期之外的结果时，测试结果为`failed`。

### 在 Intellij IDEA中运行

在使用类似Intellij IDEA这类IDE工具时，我们可以直接右键点击测试类并且选择运行`CalculatorTest`，或者我们可以使用快捷键`Ctrl+Shift+F10`(Windows系统)和`Ctrl+Shift+R`(Mac系统)来运行测试。



下图中可以查看在Intellij IDEA中执行`JUnit 5`测试通过的结果：

![Run Test In Intellij IDEA](/blog_img/translate/junit5/junit-5-getting-started/junit-5-test-result-in-intellij-idea.webp "Run Test In Intellij IDEA") 

### 在Maven中运行

若要使用`Maven`从命令行运行测试，可执行下述命令：

```bash
mvn test
```

之后将能看见类似如下的输出：

```bash
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.arhohuttunen.CalculatorTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.021 s - in com.arhohuttunen.CalculatorTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```



### 在Gradle中运行

若要使用`Gradle`从命令行运行测试，可执行下述命令：

```bash
gradle test
```

之后将能看见类似如下输出:

```bash
> Task :test
com.arhohuttunen.CalculatorTest > addNumbers() PASSED
BUILD SUCCESSFUL in 0s
```

## 总结

在这个`JUnit 5`入门示例中，我们学习了如何编写和运行简单的`JUnit 5`测试，本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-starter)中找到。