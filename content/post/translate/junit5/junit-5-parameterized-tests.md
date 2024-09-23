---
title: "[译]JUnit 5 参数化测试实用指南"
date: 2023-06-17T9:21:43+08:00
lastmod: 2023-06-17T9:21:43+08:00
draft: false
keywords: ["junit5"]
description: "JUnit5翻译专题，主要是关于JUnit5参数化测试实用指南"
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

本文翻译自[**A More Practical Guide to JUnit 5 Parameterized Tests**](https://www.arhohuttunen.com/junit-5-parameterized-tests/)。

<!--more-->

![JUnit 5 Chart Checklist](/blog_img/translate/junit5/junit-5-parameterized-tests/junit-5-checklist-clipboard.webp "JUnit 5 Chart Checklist") 

在本教程中我们将学习如何编写`JUnit5`参数化测试，本教程以结构化方式展现以便能够同时解答关于参数化测试的常见问题。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 相关视频

如果你喜欢通过视频学习，可以查看`Youtube`中相关的[**学习视频**](https://www.youtube.com/watch?v=0xSCbTYAiF0)。

## 概览

参数化测试使得可以使用不同的参数多次运行同一个测试方法，通过这种方式我们可以快速的验证不同的场景而无需为它们分别编写测试代码。

可以像编写常规`JUnit 5`测试一样编写`JUnit 5`参数化测试代码，但必须使用`@ParameterizedTest`注释，同时必须为相关测试声明参数源，可通过不同类型的参数来源注解来声明参数源。

## 单参数与@ValueSource

最简单的参数源为`@ValueSource`，它使得我们可创建一个包含原始类型(如何`short`、`byte`、`int`、`long`、`float`、`double`、`char`、`boolean`、`String`或`Class`)的数组来使用。

下述代码使用不同字符串作为测试参数：

```java
@ParameterizedTest
@ValueSource(strings = { "level", "madam", "saippuakivikauppias" })
void palindromeReadsSameBackward(String string) {
    assertTrue(StringUtils.isPalindrome(string));
}
```

顺便说下`saippuakivikauppias`在芬兰语中表示皂石供应商。

执行上述测试代码后，我们可从输出结果中看出测试方法使用不同的字符串值执行了三次。

```bash
palindromeReadsSameBackward(String)
├─ [1] level
├─ [2] madam
└─ [3] saippuakivikauppias
```

下述代码为另一个使用`int`类型进行参数化测试的示例：

```java
@ParameterizedTest
@ValueSource(ints = { 3, 6, 15})
void divisibleByThree(int number) {
    assertEquals(0, number % 3);
}
```

另一种单参数源注解是`@EnumSource`，它使用枚举类作为参数并使用枚举值进行测试：

```java
enum Protocol {
    HTTP_1_0, HTTP_1_1, HTTP_2
}

@ParameterizedTest
@EnumSource(Protocol.class)
void postRequestWithDifferentProtocols(Protocol protocol) {
    webServer.postRequest(protocol);
}
```

执行上述测试后，可发现测试方法基于`Protocol`中的每个枚举值分别执行了一次。

## 空值与@NullSource

`@ValueSource`注解不接收null值。

有一个名为`@NullSource`的特殊注解，可用来在测试中提供null参数，另一个特殊的注解是`@EmptySource`，它为`String`、`List`、`Set`、`Map`或数组提供empty值。

在下述例子中，我们将null值、空字符串和空白字符串传递给测试方法：

```java
@ParameterizedTest
@NullSource
@EmptySource
@ValueSource(strings = { " " })
void nullEmptyAndBlankStrings(String text) {
    assertTrue(text == null || text.trim().isEmpty());
}
```

也可以使用`@NullAndEmptySource`将两者结合起来。

## 多参数与@MethodSource

`@ValueSource`和`@EnumSource`注解只有在测试方法只有一个参数时生效，不过我们经常需要使用多个参数。

`@MethodSource`注解允许我们引用一个返回多参数的工厂方法，此类方法需返回`Stream`、`Iterable`、`Iterator`或参数数组。

假设我们有一个`DateUtils`类基于数字获取对应月份的名称，我们需要在参数化测试中传递多个参数，因此我们可以在工厂方法中使用`Stream`参数实现。

```java
@ParameterizedTest
@MethodSource("numberToMonth")
void monthNames(int month, String name) {
    assertEquals(name, DateUtils.getMonthName(month));
}

private static Stream<Arguments> numberToMonth() {
    return Stream.of(
            arguments(1, "January"),
            arguments(2, "February"),
            arguments(12, "December")
    );
}
```

当在`@MethodSource`注解中引用工厂方法时，它将给测试方法提供不同的`month`和`name`参数。

若在`@MethodSource`注解中没有提供方法名称，`JUnit 5`将会尝试寻找具有相同名称的方法。

```java
@ParameterizedTest
@MethodSource
void monthNames(int month, String name) {
    assertEquals(name, DateUtils.getMonthName(month));
}

private static Stream<Arguments> monthNames() {
    return Stream.of(
            arguments(1, "January"),
            arguments(2, "February"),
            arguments(12, "December")
    );
}
```

## 共享参数与@ArgumentSource

也可通过`@MethodSource`注解来引用其它类中的方法，需要使用方法的全限定名来实现。

```java
package com.arhohuttunen;

import java.util.stream.Stream;

public class StringsProvider {
    private static Stream<String> palindromes() {
        return Stream.of("level", "madam", "saippuakivikauppias");
    }
}
```

全限定名是包名称、类名称和方法名称的组合：

```java
@ParameterizedTest
@MethodSource("com.arhohuttunen.StringsProvider#palindromes")
void externalPalindromeMethodSource(String string) {
    assertTrue(StringUtils.isPalindrome(string));
}
```

另一种方式是使用实现了`ArgumentsProvider`接口的自定义类：

```java
public class PalindromesProvider implements ArgumentsProvider {
    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
        return Stream.of("level", "madam", "saippuakivikauppias").map(Arguments::of);
    }
}
```

之后再测试方法中通过`@ArgumentsSource`指定该类：

```java
@ParameterizedTest
@ArgumentsSource(PalindromesProvider.class)
void externalPalindromeMethodSource(String string) {
    assertTrue(StringUtils.isPalindrome(string));
}
```

## 多参数与@CsvSource

`@CsvSource`注解允许我们使用以逗号分隔的字符串参数，基于该注解能够以相当紧凑的方式给测试方法提供多个参数。

```java
@CsvSource({
        "Write a blog post, IN_PROGRESS, 2020-12-20",
        "Wash the car, OPENED, 2020-12-15"
})
void readTasks(String title, Status status, LocalDate date) {
    System.out.printf("%s, %s, %s", title, status, date);
}
```

如果在测试代码中写入了大量的测试数据，测试代码很快将会变得不可读，一种解决方案是通过`@CsvFileSource`注解在外部CSV文件中提供数据。

基于前面的示例，首先在tasks.csv中创建一个以逗号分隔的参数列表，将其放在`src/test/resources`目录下，文件中的每一行都是一个参数列表。

```bash
Write a blog post, IN_PROGRESS, 2020-12-20
Wash the car, OPENED, 2020-12-15
```

接下来，使用`@CsvFileSource`注解给测试方法提供测试参数。

```java
@ParameterizedTest
@CsvFileSource(resources = "/tasks.csv")
void readTasks(String title, Status status, LocalDate date) {
    System.out.printf("%s, %s, %s", title, status, date);
}
```

### 空CSV参数

如果`@CsvSource`中有empty值，`JUnit 5`会将其作为`null`值。

```java
@ParameterizedTest
@CsvSource(", IN_PROGRESS, 2020-12-20")
void nullArgument(String title, Status status, LocalDate date) {
    assertNull(title);
}
```

空字符串可使用单引号包含起来：

```java
@ParameterizedTest
@CsvSource(value = "NULL, IN_PROGRESS, 2020-12-20", nullValues = "NULL")
void customNullArgument(String title, Status status, LocalDate date) {
    assertNull(title);
}
```

如果想将null值替换为特殊的字符串，可在`@CsvSource`注解中使用`nullValues`参数：

```java
@ParameterizedTest
@CsvSource(value = "NULL, IN_PROGRESS, 2020-12-20", nullValues = "NULL")
void customNullArgument(String title, Status status, LocalDate date) {
    assertNull(title);
}
```

### 将字符串转换为其它类型

为了更好的支持类似`@CsvSource`等注解，`JUnit 5`对原始参数类型、枚举，`java.time`包中的日期和时间类型进行自动转换。

例如，这意味着它会自动将以下日期字符串转换为`LocalDate`实例:

```java
@ParameterizedTest
@ValueSource(strings = { "2018-01-01", "2018-01-31" })
void convertStringToLocalDate(LocalDate localDate) {
    assertEquals(Month.JANUARY, localDate.getMonth());
}
```

`JUnit 5`参数化测试默认支持更多类型转化，我们可查看[**JUnit5 implicit conversion**](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-argument-conversion-implicit)获取转换类型列表，而不是在此处讲解全部内容。

如果`JUnit 5`无法转化参数，它将对目标类型调用下述两种方法：
1. 只有一个`String`参数的构造方法
2. 接收一个`String`参数并返回目标类型实例的静态方法

在下面的例子中，`JUnit 5`将调用`Person`类的构造来进行`String`类型转换：

```java
public class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }
}
```

类似的，下述例子中的`Person`类也能正常工作：

```java
public class Person {
    private final String name;

    public static Person fromName(String name) {
        return new Person(name);
    }
}
```

### 自定义类型转换

若要编写自定义参数转换器，则需要实现`ArgumentConverter`接口，之后则可在任何需要自定义转换的参数上使用`@ConvertWith`注解。

举例来说，我们要写一个转换器将十六进制转化为十进制，除了实现`ArgumentConverter`，在只需要处理一种类型时我们也可以继承`TypedArgumentConverter`类：

```java
class HexConverter extends TypedArgumentConverter<String, Integer> {
    protected HexConverter() {
        super(String.class, Integer.class);
    }

    @Override
    public Integer convert(String source) throws ArgumentConversionException {
        try {
            return Integer.parseInt(source, 16);
        } catch (NumberFormatException e) {
            throw new ArgumentConversionException("Cannot convert hex value", e);
        }
    }
}
```

接下来，我们需要将测试中需要自定义转换的参数添加`@ConvertWith`注解：

```java
@ParameterizedTest
@CsvSource({
        "15, F",
        "16, 10",
        "233, E9"
})
void convertWithCustomHexConverter(int decimal, @ConvertWith(HexConverter.class) int hex){
    assertEquals(decimal, hex);
}
```

为了让测试本身技术性降低一些同时更具有可读性，我们可以进一步的创建一个元注解来封装转换：

```java
@Target({ ElementType.ANNOTATION_TYPE, ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
@ConvertWith(HexConverter.class)
public @interface HexValue {
}
```

现在，可以使用新组合的注解让测试代码更具有可读性：

```java
@ParameterizedTest
@CsvSource({
        "15, F",
        "16, 10",
        "233, E9"
})
void convertWithCustomHexConverter(int decimal, @HexValue int hex) {
    assertEquals(decimal, hex);
}
```

### 将多参数转化为对象

默认情况下，提供参数化测试的参数对应于单个方法参数，可以使用`ArgumentsAccessor`将这些参数聚合到单个测试方法参数中。

为了创建一个更具可读性和可重用性的参数聚合器，我们可编码自己实现：

```java
public class TaskAggregator implements ArgumentsAggregator {
    @Override
    public Object aggregateArguments(
            ArgumentsAccessor accessor,
            ParameterContext context
    ) throws ArgumentsAggregationException {

        return new Task(
                accessor.getString(0),
                accessor.get(1, Status.class),
                accessor.get(2, LocalDate.class)
        );
    }
}

@ParameterizedTest
@CsvSource({
        "Write a blog post, IN_PROGRESS, 2020-12-20",
        "Wash the car, OPENED, 2020-12-15"
})
void aggregateArgumentsWithAggregator(@AggregateWith(TaskAggregator.class) Task task) {
    System.out.println(task);
}
```

如同自定义参数转换器，我们也可以给聚合器创建一个速记注解：

```java
@ParameterizedTest
@CsvSource({
        "Write a blog post, IN_PROGRESS, 2020-12-20",
        "Wash the car, OPENED, 2020-12-15"
})
void aggregateArgumentsWithAnnotation(@CsvToTask Task task) {
    System.out.println(task);
}
```

现在可以在任何需要的地方使用该聚合器注解。

## 自定义测试参数名称

默认情况下，`JUnit 5`参数化测试的显示名称包括所有参数的调用索引和字符串表示形式，然而，我们可以通过`@ParameterizedTest`注解中的name属性来展示自定义的名称。

再次基于前面的月份名称示例：

```java
@ParameterizedTest(name = "{index} => number={0}, month={1}")
@MethodSource
void monthNames(int month, String name) {
    assertEquals(name, DateUtils.getMonthName(month));
}

private static Stream<Arguments> monthNames() {
    return Stream.of(
            arguments(1, "January"),
            arguments(2, "February"),
            arguments(12, "December")
    );
}
```

名称属性中的`{index}`占位符表示当前的调用序号，`{0}`,`{1}`则表示实际的参数值。

执行上述测试会得到类似如下输出：

```java
monthNames(int, String)
├─ 1 => number=1, month=January
├─ 2 => number=2, month=February
└─ 3 => number=12, month=December
```

## 总结

`JUnit 5`参数化测试允许我们消除重复的测试代码，它使得通过使用不同的参数来多次执行同一个测试方法成为可能。

* 如果我们只有一个参数在大多数情况下使用`@ValueSource`即可满足要求，我们也可以使用`@EnumSource`、`@NullSource`和`@EmptySource`
* 如果有多个参数，在大多数情况下`@MethodSource`注解是合适选择，也可以使用`@ArgumentsSource`实现重复使用
* 对于数据驱动的测试，可使用`@CsvFileSource`注解
* 可基于`ArgumentConverter`接口实现自定义参数转化规则
* 可使用`ArgumentsAggregator`接口实现参数聚合

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-parameterized-tests)中找到。
