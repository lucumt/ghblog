---
title: "[译]JUnit5 Gradle示例 - 基于Gradle运行测试"
date: 2023-06-27T14:21:43+08:00
lastmod: 2023-06-27T14:21:43+08:00
draft: false
keywords: ["junit5","gradle"]
description: "JUnit5翻译专题，主要是JUnit5 Gradle示例，基于Gradle运行测试"
tags: ["junit5","junit","java","gradle"]
categories: ["翻译","JUnit5翻译"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
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

本文翻译自[**JUnit 5 Gradle Example**](https://www.arhohuttunen.com/junit-5-gradle-example/)。

<!--more-->

![JUnit 5 Maven](/blog_img/translate/junit5/junit-5-gradle-example/junit-5-gradle.webp "JUnit 5 Maven") 

在本教程中，我们将学习在基于`Gradle`编写`JUnit 5`测试时如何获取相关的依赖，以及如何配置` JUnit Gradle`插件来运行相关测试。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 相关依赖

若要能够编写`JUnit 5`测试需要在`build.gradle`中添加junit-jupiter作为依赖项：

```groovy
dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.8.0")
}
```

之后配置让相关测试使用`JUnit`平台：

```groovy
test {
    useJUnitPlatform()
}
```

至此我们已经有了使用`Gradle`编写和运行`JUnit 5`测试的最基本配置。

## 旧版本配置

从`JUnit Jupiter 5.4.0`开始有一个聚合器组件`junit-jupiter`它可以传递对`junit-jupiter-api`、`junit-jupiter-params`和`junit-jupiter-engine`的依赖以简化依赖关系管理,这意味着我们不需要额外的依赖项就能够编写参数化测试。

为了能够使用旧版本编写`JUnit 5`测试，需要`junit-jupiter-api`组件作为依赖项，同时需要在运行时类路径下添加JUnit Jupiter测试引擎：

```groovy
dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter-api:5.3.2")
    testRuntimeOnly("org.junit.jupiter:junit-jupiter-engine:5.3.2")    
}
```

从`Gradle 4.6`开始，提供了对`JUnit Jupiter`的原生支持，而在使用`Gradle 4.5`或更早版本时，为了能够运行`JUnit 5`测试则必须配置`JUnit Gradle`插件：

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.junit.platform:junit-platform-gradle-plugin:1.3.2'
    }
}

apply plugin: 'org.junit.platform.gradle.plugin'
```

至此我们已经有了使用旧版本的`Gradle`运行`JUnit 5`测试的基本配置。

## 运行测试

`JUnit Gradle`插件默认情况下在`src/test/java`目录下查找测试用例。

可通过添加一个空测试来检查我们的配置是否生效。

```java
class GradleExampleTest {

    @Test
    void shouldRun() {

    }
}
```

在命令行中运行相关测试：

```bash
gradle test
```

我们应该能看到类似如下输出：

```bash
:test
BUILD SUCCESSFUL in 2s
```

好了，` JUnit Gradle`插件现在可正常运行我们的测试。

## 总结

在这个`JUnit 5 Gradle`教程中，我们学习了如何添加编写`JUnit 5`测试所需的依赖项以及如何配置`JUnit Gradle`插件以便能够运行测试。

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-gradle)中找到。
