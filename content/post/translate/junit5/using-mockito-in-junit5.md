---
title: "[译]在JUnit 5中使用Mockito"
date: 2023-06-19T19:21:43+08:00
lastmod: 2023-06-18T19:21:43+08:00
draft: false
keywords: ["junit5"]
description: "在JUnit 5中使用Mockito"
tags: ["junit5","java","mockito"]
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

本文翻译自[**Using Mockito With JUnit 5**](https://www.arhohuttunen.com/junit-5-mockito/)。

<!--more-->

![JUnit 5 Mockito](/blog_img/translate/junit5/using-mockito-in-junit5/junit-5-mockito.webp "JUnit 5 Mockito") 

在本教程中将学习在`JUnit 5`中如何使用`Mockito`框架，我们将以独立的方式学习测试框架，之后学习如何在`JUnit 5`中使用`Mockito`插件。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

# 相关视频

如果你喜欢通过视频学习，可以查看`Youtube`中相关的[**学习视频**](https://www.youtube.com/watch?v=p7_cTAF39A8)。

# 手工初始化

在做其它事情之前，我们需要添加`Mockito`相关的依赖：

```groovy
dependencies {
    testImplementation('org.mockito:mockito-core:3.12.4')
}
```

如果我们只是想创建一个要注入其它对象的mock，最简单的方式是调用`Mockito.mock()`方法，该方法将要实例化的对象的类作为参数。

```java
class MockitoManualTest {

    private OrderRepository orderRepository;
    private OrderService orderService;

    @BeforeEach
    void initService() {
        orderRepository = mock(OrderRepository.class);
        orderService = new OrderService(orderRepository);
    }

    @Test
    void createOrderSetsTheCreationDate() {
        Order order = new Order();
        when(orderRepository.save(any(Order.class))).then(returnsFirstArg());

        Order savedOrder = orderService.create(order);

        assertNotNull(savedOrder.getCreationDate());
    }
}
```

手工初始化在我们没有太多要mock的对象时是一个合法的解决方案。

# 基于注解初始化

# Mock自动注入

# JUnit5 Mockito插件

# 总结

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-mockito)中找到。
