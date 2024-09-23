---
title: "[译]JUnit5 整合Kotlin"
date: 2023-06-21T19:21:43+08:00
lastmod: 2023-06-21T19:21:43+08:00
draft: false
keywords: ["junit5"]
description: "JUnit5翻译专题，主要是如何在JUnit5中整合Kotlin"
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

本文翻译自[**JUnit 5 With Kotlin for Java Developers**](https://www.arhohuttunen.com/junit-5-kotlin//)。

<!--more-->

![JUnit 5 Kotlin](/blog_img/translate/junit5/junit-5-kotlin/junit-5-kotlin.webp "JUnit 5 Kotlin") 

在本教程中将学习在`Java`和`Kotlin`中编写`JUnit 5`测试的差异，同时也将学习如何在`Gradle Kotlin DSL`[^1]在构建脚本中配置`JUnit 5`环境。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 相关视频

如果你喜欢通过视频学习，可以查看`Youtube`中相关的[**学习视频**](https://www.youtube.com/watch?v=94KTm9b00VM)。

## 配置

我们使用`Groovy DSL`将传统的`Gradle`构建脚本编写为`build.gradle`文件。`Gradle Kotlin DSL`对多项改进提供了替代语法，如内容辅助和重构，通过`Kotlin DSL`编写的构建脚本被命名为`build.gradle.kts`。要在`Kotlin`中编写`JUnit 5`测试，首先需要在`build.gradle.kts`中添加`junit-jupiter`坐标并且告知其在测试中使用`JUnit`平台。

```groovy
dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.8.0")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

## 基本功能

`JUnit 5`中的大部分功能在`Kotlin`中都能像在`Java`中一样正常工作，一切都是开箱即用的。

一个显著的不同是如何在测试报告或IDE中自定义展示测试方法的名称。

在`Java`中，我们可使用`@DisplayName`注解来让方法名可读，但在`Kotlin`中我们给变量和方法添加反引号来表示。

```kotlin
@Test
fun `1 + 2 = 3`() {
    assertEquals(3, calculator.add(1, 2))
}
```

利用反引号让代码和测试结果更具有可读性，通常我们不应该使用这样的方法名称吗，但它对此需求实现很方便。

## 惰性评估

`JUnit 5`通过lambda对错误消息提供了惰性评估，利用lambda可避免构建不必要的昂贵错误消息。

在`Kotlin`中，如果一个函数的最后一个参数接收一个函数，就能重写为括号外的lambda表达式。

```kotlin
@Test
fun `1 + 2 = 3`() {
    assertEquals(3, calculator.add(1, 2)) {
        "1 + 2 should equal 3"
    }
}
```

相对于`Java`，此种实现方式让在`Kotlin`中编写断言时的语法更简洁。

## 断言

`JUnit 5`中的任何断言都能在`Kotlin`中正常工作，然而有几种`Kotlin`特定的断言方法更适合该语言，这些断言方法是`org.junit.jupiter.api`包中的顶级函数。

下面是一个代码抛出异常的断言示例，在`Java`中，我们可通过在`assertThrows()`调用中传入一个lambda表达式，在`Kotlin`中，同样可通过在断言调用后添加一个lambda表达式让其更具有可读性。

```kotlin
@Test
fun `Divide by zero should throw ArithmeticException`() {
    assertThrows<ArithmeticException> {
        calculator.divide(1, 0)
    }
}
```

这个简单的lambda语法同样适用于分组断言，通过分组断言可实现同一时刻执行多个断言并且一并反馈失败。

就像在`Java`中，我们可在`Kotlin`的`assertAll()`调用中编写lambda表达式，但该语法不太冗长。

```kotlin
@Test
fun `Square of a number should equal the number multiplied by itself`() {
    assertAll(
            { assertEquals(1, calculator.square(1)) },
            { assertEquals(4, calculator.square(2)) },
            { assertEquals(9, calculator.square(3)) }
    )
}
```

相比较`Java`，此种方式不太冗长同时更具有可读性。

## 参数化测试

在`JUnit 5`中有多种方式编写参数化测试，它们中的大部分可在`Kotlin`中无需修改直接运行。

需要考虑的一点是在使用`@MethodSource`注解时有一些不同，此注解在类中接收一个静态方法作为参数来源。

要在`Kotlin`中实现同样功能，我们需要创建一个伴生对象并给相关方法添加`@JvmStatic`注解，此注解将使该方法以`Java`静态方法的形式存在。

```java
companion object {
    @JvmStatic
    fun squares() = listOf(
            Arguments.of(1, 1),
            Arguments.of(2, 4),
            Arguments.of(3, 9)
    )
}

@ParameterizedTest(name = "Square of {0} should equal {1}")
@MethodSource("squares")
fun `Square of a number`(input: Int, expected: Int) {
    assertEquals(expected, calculator.square(input))
}
```

若没有添加`@JvmStatic`注解将会出现如下错误：

```kotlin
org.junit.platform.commons.JUnitException: Could not find method [squares] in class [com.arhohuttunen.junit5.kotlin.CalculatorParameterizedTest]
```

利用此种方式实现参数化测试虽然能正常工作但却没有`Java`中的方便，另一个需要注意的是每个类中只能有一个伴生对象，因此所有需要提供参数的方法需要组织在一起。

{{% admonition note "补充阅读" false %}}

 [**JUnit 5 参数化测试**](/post/translate/junit5/junit-5-parameterized-tests/)

{{% /admonition %}}

## 动态测试

`JUnit 5`引入了一种新的编程模型，允许我们通过`@TestFactory`注解注释的工厂方法在运行时生成动态测试。

通常我们会提供一个`DynamicTest`示例的集合，以前述的计算器为例：

```kotlin
@TestFactory
fun `Square of a number`() = listOf(
        DynamicTest.dynamicTest("Square of 1 should equal 1") {
            assertEquals(1, calculator.square(1))
        },
        DynamicTest.dynamicTest("Square of 2 should equal 4") {
            assertEquals(4, calculator.square(2))
        },
        DynamicTest.dynamicTest("Square of 3 should equal 9") {
            assertEquals(9, calculator.square(3))
        }
)
```

每个动态测试都会显示为自己的测试，但这看起来不干净且有一些重复。

再次，此处有一种方式让其更具有可读性，我们可用一些应映射函数来消除这些重复。··

`````kotlin
@TestFactory
fun `Square of a number`() = listOf(
        1 to 1,
        2 to 4,
        3 to 9
).map { (input, expected) ->
    DynamicTest.dynamicTest("Square of $input should equal $expected") {
        assertEquals(expected, calculator.square(input))
    }
}
`````

此种方式除了语法略有不同，与我们在参数化测试种的实现很相似。

## 嵌套测试

`JUnit 5`中的嵌套测试允许我们给测试定义层级结构，同`Java`中类似，我们或许希望下述代码能正常运行：

```kotlin
class NestedTest {
    @Nested
    class GetRequest {
        @Test
        fun `return existing entity`() {}
    }

    @Nested
    class PostRequest {
        @Test
        fun `create new entity`() {}
    }
}
```

上述例子会给我们一个警告:*只有非静态的嵌套类才能作为`@Nested`测试类*，`JUnit 5`找不到任何可执行的测试方法。

默认情况下，`Kotlin`中的嵌套类与`Java`中的静态类很像，只有非静态嵌套类(如内部类)才能添加`@Nested `注解作为测试类。

解决方案为将对应类在`Kotlin`中标记为`inner class`:

```kotlin
class NestedTest {
    @Nested
    inner class GetRequest {
        @Test
        fun `return existing entity`() {}
    }

    @Nested
    inner class PostRequest {
        @Test
        fun `create new entity`() {}
    }
}
```

现在`JUnit 5`能够发现内部嵌套的测试类以及对应的测试方法。

{{% admonition note "补充阅读" false %}}

 [**JUnit 5 嵌套测试**](/post/translate/junit5/junit-5-nested-tests/)

{{% /admonition %}}

## 静态方法和属性

我们已经简要介绍了静态方法和`Kotlin`，要让`Kotlin`中的方法看起来像一个`Java`静态方法，我们需要创建一个伴生对象并给对应方法上添加`@JvmStatic`注解。

```kotlin
companion object {
    @JvmStatic
    fun squares() = listOf(
            Arguments.of(1, 1),
            Arguments.of(2, 4),
            Arguments.of(3, 9)
    )
}
```

另一个可能的陷阱是在使用静态属性时，在`Java`中可通过添加`static`属性来实现，因此如果你刚接触`Kotlin`，或许会期望类似下述代码能够执行：

```kotlin
class RegisterStaticExtensionTest {
    companion object {
        @RegisterExtension
        val jettyExtension: JettyExtension = JettyExtension()
    }
}
```

但是，如果我们编写类似的代码并执行，将会看见一个关于属性为私有的错误消息:

```kotlin
org.junit.platform.commons.PreconditionViolationException: Failed to register extension via @RegisterExtension field [private static final com.arhohuttunen.junit5.kotlin.JettyExtension com.arhohuttunen.junit5.kotlin.RegisterStaticExtensionTest.jettyExtension]: field must not be private.
```

解决方案为给该属性添加`@JvmField`注解：

```kotlin
class RegisterStaticExtensionTest {
    companion object {
        @JvmField
        @RegisterExtension
        val jettyExtension: JettyExtension = JettyExtension()
    }
```

添加上该注解后会暴露该`Kotlin`属性作为一个`Java`属性并且`JUnit 5`能够发现它。

## 生命周期方法

`JUnit 5`中的生命周期方法同样能在`Kotlin`中运行。

然而，添加了`@BeforeAll`和`@AfterAll`注解的方法在默认情况下需要为静态方法，原因是`JUnit 5`为每个测试方法创建一个测试实例，并且没有其他方法可以在所有测试之间共享状态。

幸运的是，在`JUnit 5`中可通过添加`@TestInstance(Lifecycle.PER_CLASS)`注解让每个测试类只创建一个测试实例，通过对生命周期更改消除了对静态方法的要求。

```kotlin
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class LifecycleTest {
    @BeforeAll
    fun beforeAll() {
        println("Before all")
    }

    @AfterAll
    fun afterAll() {
        println("After all")
    }

    @Test
    fun firstTest() {
        println("First test")
    }

    @Test
    fun secondTest() {
        println("Second test")
    }
}
```

由于现在测试方法见共享实例状态，如果测试方法依赖于存储在实例间的状态，或许需要在`@BeforeEach`或`@AfterEach`注解方法中重置状态，一般来说，尽量避免编写依赖于这种状态的测试。

{{% admonition note "补充阅读" false %}}

 [**JUnit 5 生命周期**](/post/translate/junit5/junit-5-test-lifecycle/)

{{% /admonition %}}

## 可重复注解

当前`Kotlin`并不支持重复注解，因此使用多个扩展或标签进行测试比 Java 稍微复杂一些。

譬如，在`Java`中我们可重复添加`@Tag`注解来给某个测试添加多个标签：

```java
@Tag("first")
@Tag("second")
class RepeatableAnnotationTest {}
```

然而在`Kotlin`中我们不能同时有多个`@Tag`注解，相反必须使用`@Tags`注解来包装重复的标签。

```kotlin
@Tags(
        Tag("first"),
        Tag("second")
)
class RepeatableAnnotationTest
```

当有多个扩展时也是类似，因此我们不能直接使用多个`@ExtendWith`，相反必须使用`@Extensions`注解来包装重复的扩展。

```kotlin
@Extensions(
        ExtendWith(CoolestEverExtension::class),
        ExtendWith(SecondBestExtension::class)
)
class RepeatableAnnotationTest
```

## 总结

尽管在某些场景下有些语法与`Java`中的不同，大部分`JUnit 5`的功能特性能在`Kotlin`中完美运行，然而，由于`Kotlin`语言的工作方式，我们通常可以使代码更具可读性。

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-gradle-kotlin)中找到。

[^1]: `DSL`即`Domain Specific Language`，中文译`领域特定语言`，专门用于解决特定领域问题而设计开发的。