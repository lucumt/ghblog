---
title: "[译]JUnit 5 生命周期 - 注释之前和之后"
date: 2023-06-16T19:21:43+08:00
lastmod: 2023-06-16T19:21:43+08:00
draft: false
keywords: ["junit5"]
description: "JUnit5翻译专题，主要是关于JUnit5生命周期的博文"
tags: ["junit5","java","junit"]
categories: ["翻译","JUnit5翻译"]
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

本文翻译自[**JUnit 5 Test Lifecycle: Before and After Annotations**](https://www.arhohuttunen.com/junit-5-test-lifecycle/)。

<!--more-->

![JUnit 5 Chart Diagram](/blog_img/translate/junit5/junit-5-test-lifecycle/junit-5-chart-diagram.webp "JUnit 5 Chart Diagram") 

在本教程中我们将学习如何在测试类的每个测试方法或测试类中所有方法之前和之后运行代码，同时将了解嵌套测试和扩展测试时的执行顺序。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 生命周期相关方法

一个生命周期方法是指任何添加了`@BeforeAll`、`@AfterAll`、`@BeforeEach`或`@AfterEach`注解的方法，生命周期方法在实际测试方法之前或之后执行。

`@BeforeAll`和`@AfterAll`注解表示在测试类中使用使用该注解的方法应该在全部的测试方法之前或之后执行。

与之相似，`@BeforeEach`和`@AfterEach`分别表示使用该注解的方法应该在测试类中的每个测试方法之前或之后执行。

如果测试类中有10个测试方法，`@BeforeEach` 和`@AfterEach`会执行10次，而`@BeforeAll`和`@AfterAll`只执行一次。 

## 测试生命周期

默认情况下，`JUnit`在执行测试中的每个测试方法之前都会创建一个新的实例，这有助于我们单独运行各个测试方法并避免意外的副作用。

要了解该机制是如何工作的，可以看一下下面的示例:

```java
class PerMethodLifecycleTest {
    public PerMethodLifecycleTest() {
        System.out.println("Constructor");
    }

    @BeforeAll
    static void beforeTheEntireTestFixture() {
        System.out.println("Before the entire test fixture");
    }

    @AfterAll
    static void afterTheEntireTestFixture() {
        System.out.println("After the entire test fixture");
    }

    @BeforeEach
    void beforeEachTest() {
        System.out.println("Before each test");
    }

    @AfterEach
    void afterEachTest() {
        System.out.println("After each test");
    }

    @Test
    void firstTest() {
        System.out.println("First test");
    }

    @Test
    void secondTest() {
        System.out.println("Second test");
    }
}
```

需要注意的是使用`@BeforeAll`和`@AfterAll`注解的方法是静态方法，这是因为每个测试方法在创建新的实例时，它们之间没有共享状态。



通过下图可更直观的理解：

![JUnit 5 Per Method Lifecycle](/blog_img/translate/junit5/junit-5-test-lifecycle/junit-5-per-method-lifecycle.svg "JUnit 5 Per Method Lifecycle") 

执行上述测试类中的测试方法会得到如下输出(输出结果进行了格式化以便其看起来更直观)

```bash
Before the entire test fixture
  Constructor
    Before each test
      First test
    After each test
  Constructor
    Before each test
      Second test
    After each test
After the entire test fixture
```

观察上述输出，可发现`JUnit`会为每个测试方法构造一个测试类，生命周期相关的fixture方法只执行一次，测试相关的生命周期方法会被执行多次。

### 按类实例测试

也可以让`JUnit`在同一个测试实例中执行所有的测试方法，如果将测试类使用`@TestInstance(Lifecycle.PER_CLASS)`注解，`JUnit`将为每个测试类创建一个测试实例。



由于实例共享，此时没有必要使用静态方法：

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class PerClassLifecycleTest {
    @BeforeAll
    void beforeTheEntireTestFixture() {
        System.out.println("Before the entire test fixture");
    }

    @AfterAll
    void afterTheEntireTestFixture() {
        System.out.println("After the entire test fixture");
    }

    // ...
}
```

同样的，可以用下图理解其执行过程:

![JUnit 5 Per Class Lifecycle](/blog_img/translate/junit5/junit-5-test-lifecycle/junit-5-per-class-lifecycle.svg "JUnit 5 Per Class Lifecycle") 

此时的执行结果略有不同：

```bash
Constructor
  Before the entire test fixture
    Before each test
      First test
    After each test
    Before each test
      Second test
    After each test
  After the entire test fixture
```

从输出结果中可看出，生命周期相关方法的执行顺序没有发生改变，但不同之处在于`JUnit`只会构建一次测试类。

根本区别在于，在构造新测试类实例的默认生命周期方法中会重置实例变量中存储的状态，而当每个类的生命周期方法仅构造一次实例时，存储在实例变量中的状态则会在测试方法之间共享。

{{% admonition danger "注意" false %}}

​    如果测试方法依赖于存储在实例变量中的状态，则需要在使用`@BeforeEach` 或`@AfterEach`注解的生命周期方法中重置其状态，要尽可能的避免编写依赖与存储实例状态的测试方法。

{{% /admonition %}}

### 更具体的例子

现在我们知道了生命周期方法如何工作的，进一步探索如何在实践中使用它们是有益的。通常如果有一些计算成本较高的东西，可以在多个测试方法中进行共享。

此类示例包括打开数据库连接、从依赖注入框架获取上下文或读取文件。

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class ExpensiveResourceTest {
    private JettyServer jettyServer;

    @BeforeAll
    void startServer() throws Exception {
        jettyServer = new JettyServer();
        jettyServer.start();
    }

    @AfterAll
    void stopServer() throws Exception {
        jettyServer.stop();
    }

    @Test
    void checkServerStatus() throws IOException {
        URL url = new URL("http://localhost:8080/status");
        HttpURLConnection connection =
                (HttpURLConnection) url.openConnection();
        int response = connection.getResponseCode();

        assertEquals(200, response);
    }

    @Test
    void checkInvalidEndpoint() throws IOException {
        URL url = new URL("http://localhost:8080/invalid");
        HttpURLConnection connection =
                (HttpURLConnection) url.openConnection();
        int response = connection.getResponseCode();

        assertEquals(404, response);
    }
}
```

## 嵌套测试生命周期

生命周期方法同样可以用于嵌套测试，但在默认情况下`@BeforeAll` 和`@AfterAll`注解不会工作，这是由于嵌套类是内部类，而`Java`内部类中不支持静态方法。



可将嵌套类添加`@TestInstance(Lifecycle.PER_CLASS)`注解来实现类似效果：

```java
public class NestedLifecycleTest {
    @Nested
    class HappyPath {
        @BeforeEach
        void beforeEachHappyPath() {
            System.out.println("Before each happy path");
        }

        @AfterEach
        void afterEachHappyPath() {
            System.out.println("After each happy path");
        }

        @Test
        void happyPathOne() {
            System.out.println("Happy path one");
        }

        @Test
        void happyPathTwo() {
            System.out.println("Happy path two");
        }
    }

    @Nested
    @TestInstance(TestInstance.Lifecycle.PER_CLASS)
    class ExceptionalPath {
        @BeforeAll
        void beforeEntireExceptionalPath() {
            System.out.println("Before entire exceptional path");
        }

        @AfterAll
        void afterEntireExceptionalPath() {
            System.out.println("After entire exceptional path");
        }

        @Test
        void exceptionalPathOne() {
            System.out.println("Exceptional path one");
        }

        @Test
        void exceptionalPathTwo() {
            System.out.println("Exceptional path two");
        }
    }
}
```

如我们所见，生命周期方法分别应用在每个嵌套测试中：

```bash
Before entire exceptional path
  Exceptional path one
  Exceptional path two
After entire exceptional path
Before each happy path
  Happy path one
After each happy path
Before each happy path
  Happy path two
After each happy path
```

{{% admonition note "补充阅读" false %}}

 [**JUnit5 嵌套测试**](/post/translate/junit5/junit-5-nested-tests/)

{{% /admonition %}}

## 扩展生命周期

在使用扩展时，`JUnit`除了调用测试类的生命周期方法之外，还调用对应扩展的生命周期回调方法。

`JUnit`会确保多个注册扩展的封装行为，假设有扩展`ExtensionOne`和`ExtensionTwo`，它会确保`ExtensionOne`中所有before相关的方法都在`ExtensionTwo`之前执行，类似的，`ExtensionOne`中所有after相关的方法都在`ExtensionTwo`之侯执行。

看看如下的例子：

```java
public class ExtensionOne implements BeforeEachCallback, AfterEachCallback {
    @Override
    public void beforeEach(ExtensionContext context) {
        System.out.println("Before each from ExtensionOne");
    }

    @Override
    public void afterEach(ExtensionContext context) {
        System.out.println("After each from ExtensionOne");
    }
}

@ExtendWith(ExtensionOne.class)
@ExtendWith(ExtensionTwo.class)
public class ExtensionLifecycleTest {
    @BeforeEach
    void beforeEachTest() {
        System.out.println("Before each test");
    }

    @AfterEach
    void afterEachTest() {
        System.out.println("After each test");
    }

    @Test
    void firstTest() {
        System.out.println("First test");
    }

    @Test
    void secondTest() {
        System.out.println("Second test");
    }
}
```

执行上述代码后，可在结果中看见相关扩展的封装行为：

```bash
Before each from ExtensionOne
  Before each from ExtensionTwo
    Before each test
      First test
    After each test
  After each from ExtensionTwo
After each from ExtensionOne
Before each from ExtensionOne
  Before each from ExtensionTwo
    Before each test
      Second test
    After each test
  After each from ExtensionTwo
After each from ExtensionOne
```

{{% admonition note "补充阅读" false %}}

`JUnit5`官方文档中有关于执行顺序和扩展的更详细说明

 [**JUnit5 User Guide**](https://junit.org/junit5/docs/current/user-guide/#extensions-execution-order)

{{% /admonition %}}

## 总结

要在每次测试之前和之后执行一段代码，可使用`JUnit5`中的`@BeforeEach`和`@AfterEach`注解，类似的，若要对测试实例中的所有测试执行一次代码，可使用`@BeforeAll`和`@AfterAll`注解。

此外，可通过给测试类添加`@TestInstance(Lifecycle.PER_CLASS)`注解让其执行测试时只创建一个实例，它同时为嵌套测试中开启了"before all"和"after all"相关的生命周期方法。

最后，当注册多个扩展时，`JUnit5`会确保它们生命周期方法的封装行为。

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-lifecyle)中找到。