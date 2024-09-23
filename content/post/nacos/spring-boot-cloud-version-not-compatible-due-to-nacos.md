---
title: "由于Nacos导致的Spring Boot与Spring Cloud版本不兼容原因分析"
date: 2023-09-01T10:19:48+08:00
lastmod: 2023-09-01T10:19:48+08:00
draft: false
keywords: ["Nacos","Spring Boot","Spring Clould","版本不兼容"]
description: "简单记录由于Nacos导致的Spring Boot与Spring Cloud版本不兼容原因分析"
tags: ["nacos","spring-boot","spring-cloud"]
categories: ["java编程"]
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
centerImage: false
borderImage: true

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

简单记录由于[**Nacos**](https://nacos.io/zh-cn/)导致的`Spring Boot`与`Spring Cloud`版本不兼容的原因分析过程。

<!--more-->

在[**在Python程序中使用Nacos**](/post/nacos/integrate-python-program-to-nacos/)这篇文章中我吐槽了`Nacos`，今天我还要继续吐槽`Nacos`！

Why? 因为`Nacos`中的一个jar包引入了错误的`spring-cloud-commons`依赖，导致`Spring Boot`程序启动时出现版本不兼容问题，启动失败，浪费了我好几个小时的时间(虽然这样显得我很不专业)！

## 问题

自己准备将网关项目加入`Nacos`的服务管理和配置管理功能，依照官方文档的说明添加了`com.alibaba.cloud`相关的依赖，`pom.xml`文件中的完整配置如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.lucumt</groupId>
  <artifactId>spring-gateway-test</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>spring-gateway-demo</name>
  <url>https://lucumt.info</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring.boot.version>2.7.3</spring.boot.version>
    <spring.cloud.version>3.1.3</spring.cloud.version>
    <spring.cloud.alibaba.version>2021.1</spring.cloud.alibaba.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>2021.0.8</version>
      <type>pom</type>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-alibaba-dependencies</artifactId>
      <version>${spring.cloud.alibaba.version}</version>
      <type>pom</type>
    </dependency>
    <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
      <version>${spring.cloud.alibaba.version}</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
      <version>${spring.cloud.alibaba.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-bootstrap</artifactId>
      <version>${spring.cloud.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>${spring.boot.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <version>${spring.boot.version}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>${spring.boot.version}</version>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.24</version>
      <scope>provided</scope>
    </dependency>

  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>${spring.boot.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>

</project>
```

启动该程序后控制台提示如下错误，程序启动失败！

```powershell
Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
2023-09-01 21:02:30.144 ERROR 20760 --- [           main] o.s.b.d.LoggingFailureAnalysisReporter   : 

***************************
APPLICATION FAILED TO START
***************************

Description:

Your project setup is incompatible with our requirements due to following reasons:

- Spring Boot [2.7.3] is not compatible with this Spring Cloud release train


Action:

Consider applying the following actions:

- Change Spring Boot version to one of the following versions [2.3.x, 2.4.x] .
You can find the latest Spring Boot versions here [https://spring.io/projects/spring-boot#learn]. 
If you want to learn more about the Spring Cloud Release train compatibility, you can visit this page [https://spring.io/projects/spring-cloud#overview] and check the [Release Trains] section.
If you want to disable this check, just set the property [spring.cloud.compatibility-verifier.enabled=false]
```

## 分析

根据上述报错提示信息去[**https://spring.io/projects/spring-cloud#overview**](https://spring.io/projects/spring-cloud#overview)中去查看Release Train:

![Spring Clould Release Train](/blog_img/nacos/spring-boot-cloud-version-not-compatible-due-to-nacos/spring-cloud-release-train.png "Spring Clould Release Train") 

进一步的查看上图中标红的版本结果如下，可发现`Spring Cloud`中主要的模块版本都为`3.1.x`系列。

![Spring Clould 2021.08 Release](/blog_img/nacos/spring-boot-cloud-version-not-compatible-due-to-nacos/spring-cloud-202108-release.png "Spring Clould 2021.08 Release") 

而我自己项目中`Spring Boot`和`Spring Cloud`的版本定义如下，明明符合上图中的版本对应关系，为啥还是报错呢？

```xml
<spring.boot.version>2.7.3</spring.boot.version>
<spring.cloud.version>3.1.3</spring.cloud.version>
```

虽然可通过提示说明`spring.cloud.compatibility-verifier.enabled=false`来暂时规避掉此问题，但我还是很好奇为啥明明版本对应上了，实际上却依然报错？



遇到这种问题自己的第一反应是通过`Google`与`Stackoverflow`去查找，但并没有找到有用信息。



之后根据提示尝试将`SpringBoot`版本降低到`2.3.x`或`2.4.x`，依然未能解决问题。



没办法我只能采用终极大招[**通过二分法查找定位**](/post/other/different-ways-find-bug-in-complex-code/#通过二分查找定位)！将`Nacos`相关的依赖都去掉，然后逐步添加回去，最终发现是`spring-cloud-starter-alibaba-nacos-config`导致的，之后再次去`Google`一番，依旧没能找到原因。



怎么办？



突然间想到将日志级别设置为`DEBUG`是否能找到一些有用的信息呢？说干就干，`bootstrap.yaml`文件中添加如下配置

```yaml
logging:
  level:
    root: DEBUG
```

重新启动后控制台虽然依旧报错，但提示信息更详细，从中我们可找到如下信息:

```java
org.springframework.cloud.configuration.CompatibilityNotMetException: null
	at org.springframework.cloud.configuration.CompositeCompatibilityVerifier.verifyDependencies(CompositeCompatibilityVerifier.java:47) ~[spring-cloud-commons-3.0.1.jar:3.0.1]
	at org.springframework.cloud.configuration.CompatibilityVerifierAutoConfiguration.compositeCompatibilityVerifier(CompatibilityVerifierAutoConfiguration.java:44) ~[spring-cloud-commons-3.0.1.jar:3.0.1]
```

很明显版本不兼容是由`spring-cloud-common`导致的，在命令行执行`mvn dependency:tree  -Dincludes="org.springframework.cloud:spring-cloud-commons"`输出结果如下

```bash
[INFO] com.lucumt:spring-gateway-test:jar:1.0-SNAPSHOT
[INFO] \- com.alibaba.cloud:spring-cloud-starter-alibaba-nacos-config:jar:2021.1:compile
[INFO]    \- org.springframework.cloud:spring-cloud-commons:jar:3.0.1:compile
```

可以看出确实是由于`spring-cloud-starter-alibaba-nacos-config`包引入后导致的。



在[**Maven中央仓库**](https://mvnrepository.com/)中查看该版本对应的jar包依赖结果如下，`spring-cloud-commons`的依赖版本确实是`3.0.1`，与前述分析结果一致。

![Spring Clould Nacos依赖](/blog_img/nacos/spring-boot-cloud-version-not-compatible-due-to-nacos/spring-cloud-starter-alibaba-nacos-config-dependencies.png "Spring Clould Nacos依赖") 

## 解决

找到问题原因后，解决起来很简单，在`pom.xml`文件中手工指定`spring-cloud-commons`的依赖并下载即可：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-commons</artifactId>
    <version>${spring.cloud.version}</version>
</dependency>
```

## 反思

虽然此问题的原因和解决方案都不复杂，但我在此问题上浪费了好几个小时感觉很不划算，本着严于律己，宽以待人的原则主要问题还是出在自己身上：没有早点开启`DEBUG`日志模块，早点打开就能早发现，自己没有形成相关的问题排查方法论。

