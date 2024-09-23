---
title: "[译]在JUnit 5中使用Mockito"
date: 2023-06-19T19:21:43+08:00
lastmod: 2023-06-18T19:21:43+08:00
draft: false
keywords: ["junit5"]
description: "JUnit5翻译专题，主要是关于如何在JUnit 5中使用Mockito"
tags: ["junit5","java","mockito","junit"]
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

本文翻译自[**Using Mockito With JUnit 5**](https://www.arhohuttunen.com/junit-5-mockito/)。

<!--more-->

![JUnit 5 Mockito](/blog_img/translate/junit5/using-mockito-in-junit5/junit-5-mockito.webp "JUnit 5 Mockito") 

在本教程中将学习在`JUnit 5`中如何使用`Mockito`框架，我们将以独立的方式学习测试框架，之后学习如何在`JUnit 5`中使用`Mockito`扩展。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 相关视频

如果你喜欢通过视频学习，可以查看`Youtube`中相关的[**学习视频**](https://www.youtube.com/watch?v=p7_cTAF39A8)。

## 手工初始化

在做其它事情之前，需要添加`Mockito`相关的依赖：

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

**优点**:

*  对要mock的对象有最大程度的控制

**缺点**: 

*  代码会变得很冗长
* 无法检测框架的使用情况以及检测不正常的打桩

## 基于注解初始化

`Mockito.mock()`的一种声明式替代方案为给要mock的对象上添加`@Mock`注解，同时需要调用一个特殊方法来初始化被注解的对象。

在`Mockito 2`中有一个`MockitoAnnotations.initMocks()`方法，该方法已经被废弃并在`Mockito 3`中被`MockitoAnnotations.openMocks()`替代，该方法返回一个`AutoCloseable`对象可用于在测试之后关闭资源。

```java
public class MockitoAnnotationTest {

    @Mock
    private OrderRepository orderRepository;
    private AutoCloseable closeable;
    private OrderService orderService;

    @BeforeEach
    void initService() {
        closeable = MockitoAnnotations.openMocks(this);
        orderService = new OrderService(orderRepository);
    }

    @AfterEach
    void closeService() throws Exception {
        closeable.close();
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

`MockitoAnnotations.openMocks(this)`方法的调用可告知`Mockito`扫描测试类中任何包含有`@Mock`注解的属性并将这些属性初始化为mock对象。

**优点**:

* 容易创建mock对象
* 可读性更高

**缺点**：

* 无法检测框架的使用情况以及检测不正常的打桩

## Mock自动注入

同样可以通过给对应属性添加`@InjectMocks`注解来自动注入mock对象。

当调用`MockitoAnnotations.openMocks()`时,`Mockito`会执行如下操作：

* 对有`@Mock`注解的属性创建mock对象
* 对有`@InjectMocks`的属性创建实例并尝试将前一个步骤中创建的mock对象注入其中

使用`@InjectMocks`与我们手工创建实例所做的相同，但现在是自动化的。

```java
public class MockitoInjectMocksTests {

    @Mock
    private OrderRepository orderRepository;
    private AutoCloseable closeable;
    @InjectMocks
    private OrderService orderService;

    @BeforeEach
    void initService() {
        closeable = MockitoAnnotations.openMocks(this);
    }

    @AfterEach
    void closeService() throws Exception {
        closeable.close();
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

`Mockito`将首先尝试通过构造器注入的方式注入mock对象，之后为setter注入、属性注入。

**优点**：

* 容易注入mock对象

**缺点**:

* 无法强制使用构造器注入

{{% admonition danger "注意" false %}}

不推荐使用属性注入或setter注入，使用构造器注入时，我们可以百分百确定没有注入相关依赖之前无法实例化该类。

{{% /admonition %}}

## JUnit5 Mockito扩展

`JUnit 5`中同样有一个`Mockito`扩展让实例化更简单。

要使用该扩展，首先需要添加对应的依赖。

```groovy
dependencies {
    testImplementation('org.mockito:mockito-junit-jupiter:3.12.4')
}
```

现在可使用该扩展并避免对`MockitoAnnotations.openMocks()`的方法调用：

```java
@ExtendWith(MockitoExtension.class)
public class MockitoExtensionTest {

    @Mock
    private OrderRepository orderRepository;
    private OrderService orderService;

    @BeforeEach
    void initService() {
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

同样可以在`MockitoExtension`中使用`@InjectMocks`注解来进一步的简化配置。

```java
@ExtendWith(MockitoExtension.class)
public class MockitoExtensionInjectMocksTest {

    @Mock
    private OrderRepository orderRepository;
    @InjectMocks
    private OrderService orderService;

    @Test
    void createOrderSetsTheCreationDate() {
        when(orderRepository.save(any(Order.class))).then(returnsFirstArg());

        Order order = new Order();

        Order savedOrder = orderService.create(order);

        assertNotNull(savedOrder.getCreationDate());
    }
}
```

如果我们不像在测试用例间共享mock变量，可将mock对象注入到方法参数中。

```java
@Test
void createOrderSetsTheCreationDate(@Mock OrderRepository orderRepository) {
    OrderService orderService = new OrderService(orderRepository);
    when(orderRepository.save(any(Order.class))).then(returnsFirstArg());

    Order order = new Order();

    Order savedOrder = orderService.create(order);

    assertNotNull(savedOrder.getCreationDate());
}
```

将mock对象注入方法参数中既适用于生命周期方法也适用于测试方法本身。

**优点**：

* 不需要调用`MockitoAnnotations.openMocks()`方法
* 能检测框架使用情况以及不正确的打桩
* 容易创建mock对象
* 容易阅读

**缺点：**

* 需要一个额外的依赖`org.mockito:mockito-junit-jupiter`

## 总结

在`JUnit 5`中有3种不同的方式使用`Mockito`，前2种方式可独立于框架单独使用，而第3种方式需使用`JUnit 5`中的`Mockito`扩展。

Mock对象可通过一下方式创建和初始化:
* 通过调用`Mockito.mock()`手工创建
* 添加`@Mock`注解并通过调用`MockitoAnnotations.openMocks()`方法来初始化
* 添加`@Mock`注解并在测试中使用`MockitoExtension`扩展

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-mockito)中找到。
