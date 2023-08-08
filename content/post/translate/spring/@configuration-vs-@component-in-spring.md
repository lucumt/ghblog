---
title: "[译]Spring中@Configuration和@Component的对比"
date: 2020-01-08T16:21:26+08:00
lastmod: 2020-01-08T16:21:26+08:00
draft: false
keywords: ["spring","@Configuration","@Component"]
description: "翻译一篇老外写的博文，主要是关于Spring中@Configuration和@Component的对比"
tags: ["spring","java"]
categories: ["翻译","spring系列","java编程"]
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

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

本文翻译自[**Spring @Configuration vs @Component**](http://dimafeng.com/2015/08/29/spring-configuration_vs_component/)。

在[前一篇文章](http://dimafeng.com/2015/08/16/cglib/)中，我说过我们可以使用`@Component`作为`@Configuration`的一个备用选择，实际上这是来自于[Spring team的官方建议](https://github.com/spring-projects/spring-framework/issues/17430)

<!--more-->

> 也就是说，`@Bean`在我们不使用任何`CGLIB`代理时有一种精简模式：只需在未使用`@Configuration`注释的类上声明`@Bean`方法(但通常需要用另一种`Spring`中的stereotype实现替代，如`@Component`)，只要我们不在 @Bean 方法之间进行编程调用，程序就能正常工作。

简而言之，下面显示的每个应用程序上下文配置将以完全不同的方式来生效：

```java
@Configuration
public static class Config {

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean());
    }
}
```

```java
@Component
public static class Config {

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean());
    }
}
```

第一段代码能按照预期正常工作，`SimpleBeanConsumer`将获取一个指向`SimpleBean`单例的链接，不幸的是，该段代码[在Java数字签名环境中不生效](http://dimafeng.com/2015/08/16/cglib/)。



第二个代码则是完全不正确的，因为`Spring`会创建`SimpleBean`的单例bean，但是`SimpleBeanConsumer`会获得另一个`SimpleBean`实例，该实例不受`Spring`上下文控制。

此现象的原因解释如下：

如果使用`@Configuration`，则所有标记为`@Bean`的方法都将被包装到`CGLIB`包装器中，该包装器的工作方式为该方法第一次调用时将执行原始方法的主体，并将生成的对象注册在`Spring`上下文中，之后对该方法的调用都只返回从`Spring`上下文中获取到的bean。

在上面的第二个代码块中，`new SimpleBeanConsumer(simpleBean())`只是调用一个纯java方法调用，要更正该代码块，可修改如下：

```java
@Component
public static class Config {
    @Autowired
    SimpleBean simpleBean;

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean);
    }
}
```

这篇文章的所有代码示例都可以在我的[个人GitHub](https://github.com/dimafeng/dimafeng-examples/tree/master/spring-config)中找到。