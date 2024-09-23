---
title: "[译]JUnit5 Maven示例-基于Surefire运行测试"
date: 2023-06-23T12:21:43+08:00
lastmod: 2023-06-23T12:21:43+08:00
draft: false
keywords: ["junit5","maven"]
description: "JUnit5翻译专题，主要是JUnit5 Maven示例-基于Surefire运行测试"
tags: ["junit5","junit","java","maven"]
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

本文翻译自[**JUnit 5 Maven Example: Running Tests with Surefire**](https://www.arhohuttunen.com/junit-5-maven-example/)。

<!--more-->

![JUnit 5 Maven](/blog_img/translate/junit5/junit-5-maven-example/junit-5-maven.webp "JUnit 5 Maven") 

在本教程中，我们将学习在基于`Maven`编写`JUnit 5`测试时如何获取相关的依赖，以及如何配置`maven-surefire-plugin`来运行相关测试。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 相关依赖

若要能够编写`JUnit 5`测试需要在`pom.xml`中添加`junit-jupiter`作为依赖项：

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.8.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

此外还需添加`maven-surefire-plugin`以便能运行`JUnit 5`测试:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.2</version>
        </plugin>
    </plugins>
</build>
```

至此我们已经有了使用`Maven`编写和运行`JUnit 5`测试的最基本配置。

## 旧版本配置

从`JUnit Jupiter 5.4.0`开始，有一个聚合器组件`junit-jupiter`，它可以传递对`junit-jupiter-api`、`junit-jupiter-params`和`junit-jupiter-engine`的依赖关系，从而简化依赖关系管理。

为了能够使用旧版本编写`JUnit 5`测试，需要`junit-jupiter-api`组件作为依赖项：

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.3.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

从`Maven Surefire2.22.0` 开始，对`JUnit Jupiter`提供了原生支持，当使用`Maven Surefire2.21.0`或更早版本时，则必须使用`Maven Surefire`插件来运行测试：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.21.0</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>1.3.2</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

同样需要在运行时类路径下添加`JUnit Jupiter`测试引擎，可在` maven-surefire-plugin`插件中添加相关依赖：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.21.0</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>1.3.2</version>
                </dependency>
                <dependency>
                    <groupId>org.junit.jupiter</groupId>
                    <artifactId>junit-jupiter-engine</artifactId>
                    <version>5.3.2</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

至此我们已经有了使用旧版本的`Maven Surefire`运行`JUnit 5`测试的基本配置。

## 运行测试

` maven-surefire-plugin`插件默认情况下在`src/test/java`目录下查找测试用例。

可通过添加一个空测试来检查我们的配置是否生效。

```java
class MavenExampleTest {

    @Test
    void shouldRun() {

    }
}
```

在命令行中运行相关测试：

```bash
mvn test
```

我们应该能看到类似如下输出：

```bash
[INFO] --- maven-surefire-plugin:2.22.0:test (default-test) @ junit5-maven ---
[INFO]
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running MavenExampleTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.003 s - in MavenExampleTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

好了，`Maven Surefire`现在可正常运行测试。

## 总结

在这篇`JUnit 5 Maven` 示例中，我们学习了如何添加编写`JUnit 5`测试所需的依赖项以及如何配置`Maven Surefire`插件以便能够运行测试。

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-maven)中找到。