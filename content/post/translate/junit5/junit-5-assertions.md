---
title: "[译]JUnit 5 断言 - 验证测试结果"
date: 2023-05-01T13:37:22+08:00
lastmod: 2023-05-01T13:37:22+08:00
draft: false
keywords: ["junit5","assert"]
description: "JUnit5翻译专题，主要是关于JUnit 5中断言方法的使用"
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

本文翻译自[**JUnit 5 Assertions: Verifying Test Results**](https://www.arhohuttunen.com/junit-5-assertions/)。

<!--more-->

![JUnit 5 Assertions](/blog_img/translate/junit5/junit-5-assertions/junit-5-checklist-with-computer.webp "JUnit 5 Assertions") 

## 概览

在本文中，我们将学习如何通过`JUnit 5`断言来验证测试结果，我们将学习断言的基本方法吗、如何自定义错误消息，以及如何将多个断言作为一个分组运行。



本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 断言

`JUnit 5`断言让测试结果与预期结果的验证变得更容易，只要一个测试中有断言失败，整个测试就会失败。类似的，只有单个测试中所有的断言都通过，测试才能通过。



`JUnit 5`中的断言是`org.junit.jupiter.api.Assertions`中的静态方法，下面我们将会详细了解这些方法的使用场景。

### 值比较

在验证结果时，一个最常见的场景是我们希望预期结果与实际结果相等，`JUnit 5`提供了`assertEquals()`与`assertNotEquals()`方法来对值进行相等性和不等性比较。



在这个例子中，我们有一个简单的`Calculator`类用于将两数相加，我们希望计算结果是准确的：

```java
@Test
void addNumbers() {
    Calculator calculator = new Calculator();
    assertEquals(3, calculator.add(1, 2));
}
```

如果断言失败，我们可以在错误消息中同时看到错误值和期望值：

```bash
org.opentest4j.AssertionFailedError:
Expected :3
Actual   :2
```

### 布尔值

通常，当我们希望返回的值为true或者false，可通过`assertEquals()`方法实现，但`JUnit 5`中提供了更简洁的`assertTrue()`与`assertFalse()`方法来实现此功能。



在下面例子中我们可验证一个人的名字是否以特定字母开头：

```java
@Test
void firstNameStartsWithJ() {
    Person person = new Person("John", "Doe");
    assertTrue(person.getFirstName().startsWith("J"));
}
```

类似的，若要断言返回值不为true，可通过`assertFalse()`实现。

### 空值

有时候我们希望一个对象为空或者非空，要实现此目的，可通过`JUnit 5`中的断言方法`assertNull()`和`assertNotNull()`实现。



在下面的例子中我们可验证一个人的名字是否不为空：

```java
@Test
void personHasFirstName() {
    Person person = new Person("John", "Doe");
    assertNotNull(person.getFirstName());
}
```

如果断言失败，我们将会看到类似如下的报错信息：

```bash
org.opentest4j.AssertionFailedError: expected: not <null>
```

尽管有时候可能需要断言空值，但我们通常应该在程序中避免返回空值。



{{% admonition note "补充阅读" false %}}

[**避免不必要的空值检测**](https://www.arhohuttunen.com/avoiding-unnecessary-null-checks/)

{{% /admonition %}}

### 迭代器

有时候我们需要验证一个集合中包含我们期望的元素，例如，我们可能想验证自己的排序算法是否有效。



`JUnit 5`中的`assertIterableEquals()`可用于验证一个迭代器对象是否包含我们期望的元素，我们可以比较任何实现了`Iterable`接口的类。



在下面例子中我们可验证一个list在排序后元其素顺序是否正确：

```java
@Test
void iterablesEqual() {
    final List<String> list = Arrays.asList("orange", "mango", "banana");
    final List<String> expected = Arrays.asList("banana", "mango", "orange");

    Collections.sort(list);

    assertIterableEquals(expected, list);
}
```

假如我们的排序算法没生效，没有对迭代器对象进行排序，则断言会失败并显示一条错误信息：

```bash
org.opentest4j.AssertionFailedError: iterable contents differ at index [0],
Expected :<banana>
Actual   :<orange>
```

`assertIterableEquals()`同样可用来检查迭代器对象的长度是否匹配，如果我们添加1个对象到迭代器中，断言结果会失败并显示出另外一条错误信息：

```bash
org.opentest4j.AssertionFailedError: iterable lengths differ,
Expected :<3>
Actual   :<4>
```

> **两个迭代器只有在它们全部为空或包含相同的值时才相等。**

### 数组

断言数组与断言迭代器很类似，可通过`JUnit 5`中的`assertArrayEquals()`方法实现:

```java
@Test
void arraysEqual() {
    final int[] array = { 3, 2, 1 };
    final int[] expected = { 1, 2, 3 };

    Arrays.sort(array);

    assertArrayEquals(expected, array);
}
```

> **两个数组只有在都为空或者包含相同的元素时才相等[^1]**

### 值对象

在断言两个对象是否相等时，我们需要考虑一些事项。



在下面的例子中有一个`Person`类包含姓和名，我们想比较两个`Person`对象是否相同：

```java
@Test
void personsAreSame() {
    Person john = new Person("John", "Doe");
    Person doe = new Person("John", "Doe");

    assertEquals(john, doe);
}
```

运行该测试后，会提示测试失败并显示出一条相当神秘的错误消息：

```bas
org.opentest4j.AssertionFailedError:
Expected :com.arhohuttunen.junit5.assertions.Person@eec5a4a
Actual   :com.arhohuttunen.junit5.assertions.Person@2b2948e2
```

预期的对象和实际对象包含相同的属性值，但`assertEquals()`仍然失败，发生了什么事情？



原因为`Java`中的相等性使用`equals()`方法进行比较实现,`equals()`方法的默认实现是检查两个对象是否引用相同的对象，由此导致断言测试失败。

{{% admonition note "补充阅读" false %}}

[**Java hashCode() and equals()**](https://howtodoinjava.com/java/basics/java-hashcode-equals-methods/)

{{% /admonition %}}



要解决此问题，我们需要自己实现`equals()`方法来比较类中的属性，在重写`equals()`方法后，需要一并重写`hashCode()`方法：

```java
public class Person {

    // ...

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return firstName.equals(person.firstName) &&
                lastName.equals(person.lastName);
    }

    @Override
    public int hashCode() {
        return Objects.hash(firstName, lastName);
    }
}
```

重新运行测试，可发现测试执行通过，重写`equals()`方法后在比较对象是否相等时通过属性比较实现。

### 引用对象

有时我们想确保两个对象指向或者不指向同一个实例，例如，要验证某个方法返回的是对象副本而不是相同的对象,`JUnit 5`提供了`assertSame()`和`assertNotSame()`方法来实现此功能：

```java
@Test
void personsAreNotSameInstance() {
    Person john = new Person("John", "Doe");
    Person doe = new Person("John", "Doe");

    assertNotSame(john, doe);
}
```

即使两个对象具有相同的值，该示例也会如我们所期望的那样测试通过，因为它们是两个单独的实例吗，如果测试失败，我们将看到类似如下的错误消息:

```bash
org.opentest4j.AssertionFailedError: expected: not same but was: <Person{firstName='John', lastName='Doe'}>
Expected :not same
Actual   :<Person{firstName='John', lastName='Doe'}>
```

### 异常

要确保程序中的错误处理能正常工作，我们可以验证一段代码在某些条件下是否抛出特定的异常，这可以通过`JUnit 5`中的`assertThrows()`方法来实现：

```java
@Test
void divideByZeroThrowsIllegalArgumentException() {
    Calculator calculator = new Calculator();
    assertThrows(IllegalArgumentException.class, () -> calculator.divide(1, 0));
}
```

在上面例子中， 如果我们试图将0作为除数，程序将抛出`throw IllegalArgumentException`。



如果没有抛出异常，测试将不通过同时显示出一条错误信息：

```bash
org.opentest4j.AssertionFailedError: Expected java.lang.IllegalArgumentException to be thrown, but nothing was thrown.
```

类似的，如果程序抛出一个非预期的异常，测试同样会不通过并显示出一条不同的错误消息：

```bash
org.opentest4j.AssertionFailedError: Unexpected exception type thrown ==>
Expected :<java.lang.IllegalArgumentException>
Actual   :<java.lang.ArithmeticException>
```

在某些情况下，我们想要验证有关异常的信息，例如错误消息或原因，在这种情况下，我们可以捕获抛出的异常：

```java
@Test
void divideByZeroThrowsIllegalArgumentException() {
    Calculator calculator = new Calculator();
    Throwable thrown = assertThrows(IllegalArgumentException.class, () -> calculator.divide(1, 0));
    assertEquals("Cannot divide by zero", thrown.getMessage());
}
```

### 超时

有时候我们想确保程序执行时间不能超过某个限制，此时我们可用`assertTimeout()`或`assertTimeoutPreemptively()`来实现。

这两个方法的不同点在于`assertTimeout()`与调用者在同一个线程中执行，并且即使超时也不会中止，而`assertTimeoutPreemptively()`则与调用者在不同的线程中，在超时后会中止。



前面的意思是第一种方式的测试将一直执行下去，而第二种方式测试如果超过超时程序就会立即停止。



让我们看一个例子：

```java
@Test
void returnValueBeforeTimeoutExceeded() {
    final String message = assertTimeout(Duration.ofMillis(50), () -> {
        Thread.sleep(100);
        return "a message";
    });
    assertEquals("a message", message);
}
```

由于执行时间将超过限制，我们会看到一条错误信息：

```bash
org.opentest4j.AssertionFailedError: execution exceeded timeout of 100 ms by 50 ms
```

如果我们想中止执行，可通过调用`assertTimeoutPreemptively()`方法来实现：

```java
@Test
void abortWhenTimeoutExceeded() {
    final String message = assertTimeoutPreemptively(Duration.ofMillis(50), () -> {
        Thread.sleep(100);
        return "another message";
    });
    assertEquals("another message", message);
}
```

此时如果程序执行时间超时，我们将会看到一条略有不同的错误消息：

```bash
org.opentest4j.AssertionFailedError: execution timed out after 50 ms
```

这里的区别在于执行在超时处停止程序运行。

## 自定义错误信息

为`JUnit 5`断言提供自定义错误消息很容易，所有断言方法都有一个可选的错误消息作为最后一个参数：

```java
@Test
void addNumbers() {
    Calculator calculator = new Calculator();
    assertEquals(3, calculator.add(1, 2), "1 + 2 should equal 3");
}
```

自定义错误消息不会替换默认错误消息，相反，断言失败时会将自定义消息添加到错误消息之前：

```bash
org.opentest4j.AssertionFailedError: 1 + 2 should equal 3 ==>
Expected :3
Actual   :-1
```

在某些情况下，我们需要构建一些更复杂的错误消息，此时我们可将错误消息作为lambda表达式中的最后一个参数：

```java
@Test
void addingEmployeesToPersonnel() {
    Person employee = new Person("John", "Doe");

    Set<Person> personnel = new HashSet<>();
    personnel.add(employee);

    assertTrue(personnel.contains(employee),
            () -> String.format("Personnel file for %s was not found", employee));
}
```

上述例子中的错误消息不是那么复杂，但是，通过使用此种方式`JUnit 5`只会在断言失败时才构造错误消息，我们只需在程序运行失败时耗费相应的计算成本。

## 分组断言

在执行测试时，测试程序将在第一次断言失败时终止，而利用`JUnit 5`中的分组断言，我们可以在反馈失败之前运行完所有的断言测试，可以通过使用`assertAll()`方法并提供不同的断言作为该方法的参数来实现此功能。



假设我们想要验证一个人的名字是否正确，这意味着我们需要验证姓和名都是正确的：

```java
@Test
void firstAndLastNameMatches() {
    Person person = new Person("John", "Doe");

    assertAll("person"
            () -> assertEquals("John", person.getFirstName()),
            () -> assertEquals("Doe", person.getLastName())
    );
}
```

如果上述例子执行失败，所有的断言在程序失败并统一反馈错误消息之前都会被执行：

```bash
org.opentest4j.MultipleFailuresError: person (2 failures)
	expected: <John> but was: <Jane>
	expected: <Doe> but was: <Woodlawn>
```

可以看到，程序报告了所有的错误并让错误修复变得更容易。

> **单个测试只能有一条原因导致测试失败，我们不应该试图通过在单个测试中验证多个条件来减少测试数量，但在某些场景下，当断言在语义上密切相关时，我们可在单个测试中添加多个断言。**

## 高级匹配

虽然`JUnit 5`中的断言足以满足许多测试场景，但有时我们需要更强的选项。例如验证一个列表是否为特定大小、列表是否包含具有特定属性值的元素、列表是否已排序并包含特定的元素等，我们可以自己编写代码逻辑来实现，但更好的方式是断言库替我们实现。



此时`JUnit 5`断言是不够的，因此`JUnit 5`文档建议我们在这种场景下使用第三方断言库，其中最流行的是`Hamcrest`、`AssertJ`和`Truth`。



我们不准备在此教程中介绍这些库的详细信息，但我们可以快速看一下这些类库中的一些断言方法是如何使用的。

### Hamcrest

`Hamcrest`是它们中最古老的一个，在下面的例子中我们想验证一个列表是否只包含一个元素，在`JUnit 5`中可通过如下方式实现：

```java
@Test
void listHasOneItem() {
    List<String> list = new ArrayList();
    list.add("Hello");
    assertEquals(list.size(), 1);
}
```

这段代码看起来也不是那么差，可以看下`Hamcrest`的替代方案，可以通过向断言方法传递一个匹配器方法作为参数来编写断言：

```java
@Test
void listHasOneItem() {
    List<String> list = new ArrayList();
    list.add("Hello");
    assertThat(list, hasSize(1));
}
```

阅读此段代码，看起来更流畅，更接近自然语言，但我们可能会争论说第一个例子的可读性足够好，也许我们还没被说服。

### AssertJ

接下来，我们快速浏览下`AssertJ`,`Hamcrest`和`AssertJ`最主要的区别是`Hamcrest`依赖于匹配器方法，而在`AssertJ`中我们可以进行链式方法调用。



如果我们想知道一个列表是否包含具有特定属性值的元素该怎么办？先只用`JUnit 5`编写测试代码：

```java
@Test
void listHasPerson() {
    List<Person> peopleaa = new ArrayList<>();
    people.add(new Person("John", "Doe"));
    people.add(new Person("Jane", "Doe"));
    assertTrue(people.stream().anyMatch(p -> p.getFirstName().equals("John")));
}
```

嗯，看起来不是那么美观，另外，如果我们的测试代码逻辑出现错误怎么办？



看下使用`AssertJ`断言是如何实现的：

```java
@Test
void listHasPerson() {
    List<Person> people = new ArrayList<>();
    people.add(new Person("John", "Doe"));
    people.add(new Person("Jane", "Doe"));
    assertThat(people).extracting("firstName").contains("John");
}
```

很容易发现这种方式可读性更好，同时我们也在测试代码中移除了容易出错的代码逻辑。

### Truth

最后，我们来看下`Truth`,它和`AssertJ`很像，最显著的差异是`Truth`试图提供更简单的API，而`AssertJ`则有一系列更复杂的断言方法。



看下第三个示例，我们想验证一个列表中是否来排序并且包含特定的元素，下面的代码是用`Truth`实现的效果：

```java
@Test
void listHasItemsInOrder() {
    List<String> fruits = new ArrayList<>();
    fruits.add("Citron");
    fruits.add("Orange");
    fruits.add("Grapefruit");
    assertThat(fruits).containsExactly("Citron", "Grapefruit", "Orange").inOrder();
}
```

再一次，变得更简洁和更容易阅读。

## 总结

`JUnit 5`中的断言方法让预期结果与实际结果的验证变得更容易：

* `JUnit 5`中的断言是位于`org.junit.jupiter.api.Assertions`类中的静态方法

* 失败的断言会在错误消息中显示预期值和实际值
* 为了展示更多断言失败的信息，可在每个断言方法中传递一个自定义错误信息
* 利用`assertAll()`方法可对断言进行分组，先执行里面的断言，然后统一返回错误信息
* 对于更复杂的断言，`JUnit 5`官方文档建议使用第三方断言库，如`Hamcrest`、`AssertJ`或`Truth`



本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-assertions)中找到。

[^1]: `assertArrayEquals()`不关注元素顺序，而`assertIterableEquals()`关注元素顺序是否相同。