---
title: "[译]JUnit 5 嵌套测试 - 将关联的测试分组在一起"
date: 2023-05-02T13:12:24+08:00
lastmod: 2023-05-02T13:12:24+08:00
draft: false
keywords: ["junit5"]
description: "JUnit5翻译专题，主要是翻译关于JUnit 5中关联测试的实现与使用"
tags: ["junit5","java","junit"]
categories: ["翻译","JUnit5翻译"]

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

本文翻译自[**JUnit 5 Nested Tests: Grouping Related Tests Together**](https://www.arhohuttunen.com/junit-5-nested-tests/)。

<!--more-->

![JUnit 5 Brackets Hub](/blog_img/translate/junit5/junit-5-nested-tests/junit-5-brackets_hub.webp "JUnit 5 Brackets Hub") 

在本教程中我们讲学习如何利用`JUnit 5`编写嵌套测试，学习如何提供一个层级结构用于描述测试方法间的依赖关系。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。


如果你喜欢通过视频学习，可以查看`Youtube`中相关的[**学习视频**](https://www.youtube.com/watch?v=TsCew_Oq6uE)

## 无嵌套测试

为了给嵌套测试奠定基础，首先看一个没有嵌套的例子，为了简洁起见省略了测试内容，完整的内容在`GitHub`中。

```java
public class MoneyTest {
    @Test
    @DisplayName("monies with same amounts and currency are equal")
    void moniesWithSameAmountsAndCurrencyAreEqual() {
        CurrencyUnit eur = CurrencyUnit.of("EUR");
        Money first = Money.of(eur, 3.99);
        Money second = Money.of(eur, 3.99);

        assertEquals(second, first);
    }

    @Test
    @DisplayName("monies with different amounts are not equal")
    void moniesWithDifferentAmountsAreNotEqual() { }

    @Test
    @DisplayName("monies with different currencies are not equal")
    void moniesWithDifferentCurrenciesAreNotEqual() { }

    @Test
    @DisplayName("can add monies of same currency")
    void addMoneyWithSameCurrency() { }

    @Test
    @DisplayName("cannot add monies of different currency")
    void addMoneyWithDifferentCurrency() { }
}
```

此处我们有一个包含多个用于测试`Money`类的测试方法，`Money`中包含货币与金额，可检测`Money`实例是否相等，也可以对其相加。



当在命令行运行上述代码时，下面展示了大致的输出结果：

```bash
MoneyTest > monies with same amounts and currency are equal PASSED
MoneyTest > can add monies of same currency PASSED
MoneyTest > cannot add monies of different currency PASSED
MoneyTest > monies with different amounts are not equal PASSED
MoneyTest > monies with different currencies are not equal PASSED
```

此处没有太多的测试方法，所以仍然具有较好的可读性，然而，我们已经能够看出关联的测试方法在输出结果中没有分组在一起。

## 添加嵌套类

`JUnit 5`中的嵌套测试为我们提供了一种构建层级结构的实现，可基于逻辑结构组织测试，组织起来的嵌套结构能让我们更好的表述测试方法间的关系。



通过在测试类中创建内部类并添加`@Nested`注解，我们可在`JUnit 5`中创建嵌套测试，同样可通过添加`@DisplayName`注解给嵌套类赋予一个更容易阅读的名称。

```java
public class MoneyTest {
    @Nested
    @DisplayName("equality is based on values")
    class Equality {
        @Test
        @DisplayName("monies with same amounts are equal")
        void moniesWithSameAmountsAreEqual() { }

        @Test
        @DisplayName("monies with different amounts are not equal")
        void moniesWithDifferentAmountsAreNotEqual() { }

        @Test
        @DisplayName("monies with different currencies are not equal")
        void moniesWithDifferentCurrenciesAreNotEqual() { }
    }

    @Nested
    @DisplayName("adding monetary amounts")
    class Addition {
        @Test
        @DisplayName("can add monies of same currency")
        void addMoneyWithSameCurrency() { }

        @Test
        @DisplayName("cannot add monies of different currency")
        void addMoneyWithDifferentCurrency() { }
    }
}
```

这个例子没有特别之处，但我们已经将检查金额相等和金额累加的测试关注点分离开来了，该测试代码有更多层级结构，同时相关联的测试方法被更好的分组在一起。



在命令行中重置执行后，可以看见与之前的一些差异：

```bash
MoneyTest > adding monetary amounts > can add monies of same currency PASSED
MoneyTest > adding monetary amounts > cannot add monies of different currency PASSED
MoneyTest > equality is based on values > monies with different amounts are not equal PASSED
MoneyTest > equality is based on values > monies with different currencies are not equal PASSED
MoneyTest > equality is based on values > monies with same amounts are equal PASSED
```

与没有嵌套的测试相比，由于相关的测试被分组在一起，所以组织的更好一些，同代码类似，输出结果有更多的结构。



从IDE工具中运行后能看见添加嵌套结构后带来的额外好处，下图展示了在Intellij IDEA中运行后的结果：

![JUnit 5 Nested Test In Intellij IDEA](/blog_img/translate/junit5/junit-5-nested-tests/junit-5-nested-test-intellij-idea.webp "JUnit 5 Nested Test In Intellij IDEA") 

<figcaption>在Intellij IDEA中运行嵌套测试</figcaption>

该报告看起来已经要好一些，每个嵌套类都可被展开或折叠，可只关注于想要的结果。

## 添加二级嵌套

来看一个更复杂的示例，我们正在测试一个REST controller，我们正在测试用于创建、读取和删除产品的`HTTP` `POST`、`GET`和`DELETE`方法，我们会检查验证各种内容，如请求正文字段和`HTTP`响应状态码。

```java
class ProductControllerTest {
    @Test
    @DisplayName("POST returns HTTP status Bad Request when fields are missing")
    void postReturnsHttpStatusBadRequestWhenFieldsAreMissing() throws Exception {
        Product product = new Product(null, null, null);

        mockMvc.perform(post("/product")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(product)))
                .andExpect(status().isBadRequest());
    }

    @Test
    @DisplayName("POST does not create a product when fields are missing")
    void postDoesNotCreateProductWhenFieldsAreMissing() throws Exception { }

    @Test
    @DisplayName("POST returns HTTP status Created when fields are valid")
    void postReturnsHttpStatusCreatedWhenFieldsAreValid() throws Exception { }

    @Test
    @DisplayName("GET returns HTTP status Not Found when product is not found")
    void getReturnsHttpStatusNotFoundWhenProductIsNotFound() throws Exception { }

    @Test
    @DisplayName("GET returns HTTP status OK when product is found")
    void getReturnsHttpStatusOkWhenProductIsFound() throws Exception { }

    @Test
    @DisplayName("GET returns found product as JSON when product is found")
    void getReturnsFoundProductAsJsonWhenProductIsFound() throws Exception { }

    @Test
    @DisplayName("DELETE returns HTTP status Not Found when product is not found")
    void deleteReturnsHttpStatusNotFoundWhenProductIsNotFound() throws Exception { }

    @Test
    @DisplayName("DELETE returns HTTP status No Content when product is found")
    void deleteReturnsHttpStatusNoContentWhenProductIsFound() throws Exception { }
}
```

这些测试方法都有自定义名称，但描述信息和方法名称变的很长，我们甚至可以说已经变的很冗长。

在命令行中执行后会出现同样的问题：

```java
ProductControllerTest > POST returns HTTP status Created when fields are valid PASSED
ProductControllerTest > DELETE returns HTTP status No Content when product is found PASSED
ProductControllerTest > GET returns HTTP status OK when product is found PASSED
ProductControllerTest > POST does not create a product when fields are missing PASSED
ProductControllerTest > GET returns HTTP status Not Found when product is not found PASSED
ProductControllerTest > DELETE returns HTTP status Not Found when product is not found PASSED
ProductControllerTest > GET returns found product as JSON when product is found PASSED
ProductControllerTest > POST returns HTTP status Bad Request when fields are missing PASSED
```

我们可看出，随着测试数量增加，输出内容开始变得有点难以阅读，虽然语言本身很容易阅读，但输出结果变得冗长，同时难以阅读。



现在我们可以看下如果在`HTTP`方法中添加嵌套、并为假设的场景添加另外一层嵌套(如产品存在/不存在)时会发生什么。

```java
class ProductControllerTest {
    @Nested
    @DisplayName("Creating a product")
    class Post {
        @Nested
        @DisplayName("when fields are missing")
        class WhenFieldsAreMissing {
            @Test
            @DisplayName("return HTTP status Bad Request")
            void returnHttpStatusBadRequest() throws Exception { }

            @Test
            @DisplayName("do not create a product")
            void doNotCreateProduct() throws Exception { }
        }

        @Nested
        @DisplayName("when fields are valid")
        class WhenFieldsAreValid {
            @Test
            @DisplayName("return HTTP status Created")
            void returnHttpStatusCreated() throws Exception { }
        }
    }

    @Nested
    @DisplayName("Finding a product")
    class GetById {
        @Nested
        @DisplayName("when product is not found")
        class WhenProductIsNotFound {
            @Test
            @DisplayName("return HTTP status Not Found")
            void returnHttpStatusNotFound() throws Exception { }
        }

        @Nested
        @DisplayName("when product is found")
        class WhenProductIsFound {
            @Test
            @DisplayName("return HTTP status OK")
            void returnHttpStatusOk() throws Exception { }

            @Test
            @DisplayName("return found product as JSON")
            void returnFoundProductAsJson() throws Exception { }
        }
    }

    @Nested
    @DisplayName("Deleting a product")
    class Delete {
        @Nested
        @DisplayName("when product is not found")
        class WhenProductIsNotFound {
            @Test
            @DisplayName("return HTTP status Not Found")
            void returnHttpStatusNotFound() throws Exception { }
        }

        @Nested
        @DisplayName("when product is found")
        class WhenProductIsFound {
            @Test
            @DisplayName("return HTTP status No Content")
            void returnHttpStatusNoContent() throws Exception { }
        }
    }
}
```

测试代码看起来更长，但我们也能发现描述信息和方法名称变的更简洁。



现代IDE允许折叠和展开代码块，因此我们可以展开`Post`类和`Delete`类，可以只查看`GeetById`类。



通过IDE执行测试能够展示出测试结果有多少层级，下图为Intellij IDEA中的运行结果：

![JUnit 5 Nested Test In Intellij IDEA](/blog_img/translate/junit5/junit-5-nested-tests/junit-5-nested-test-intellij-idea-1.webp "JUnit 5 Nested Test In Intellij IDEA") 

<figcaption>在Intellij IDEA中运行嵌套测试</figcaption>



在命令行中执行后，可发现输出结构中有更多的结构：

```bash
ProductControllerTest > Deleting a product > when product is found > return HTTP status No Content PASSED
ProductControllerTest > Deleting a product > when product is not found > return HTTP status Not Found PASSED
ProductControllerTest > Finding a product > when product is found > return found product as JSON PASSED
ProductControllerTest > Finding a product > when product is found > return HTTP status OK PASSED
ProductControllerTest > Finding a product > when product is not found > return HTTP status Not Found PASSED
ProductControllerTest > Creating a product > when fields are valid > return HTTP status Created PASSED
ProductControllerTest > Creating a product > when fields are missing > return HTTP status Bad Request PASSED
ProductControllerTest > Creating a product > when fields are missing > do not create a product PASSED
```

输出结果受益于嵌套，相关联的结果更容易被找到，如果我们设法创建一个经过深思熟虑的结构，它就像导航结果的面包屑一样便于使用。

## 避免陷阱

如果使用得当，`JUnit 5`嵌套测试可以成为一个强有力的工具，但是同其它的工具一样，使用嵌套测试也会伴随着一些陷阱。

### 忽略代码异味

当我们有编写嵌套测试的冲动时，我们应该问下自己为什么要这样做，如果测试类变得越来越大并且需要进行组织，或许意味着测试类正在做太多的事情。



我们应该问自己这些问题：

* 是否能够将任一嵌套类从测试类中抽取出来让它们只关注自身逻辑？
* 嵌套类中的任一假定测试是否意味着其在做不止一件事情？



相对于添加更多的结构来测试，我们应该考虑是否有必要进行重构，添加层级结构也会增加测试的复杂性，应该尽可能的避免添加复杂性。

### 试图消除重复

许多教程建议我们通过在`@BeforeEach`方法中构造共享对象并将它们定义为类成员变量来消除重复，这个建议的初衷是好的，但在测试代码中消除重复有更多的讲究。



我们仔细看下之前的产品controller测试：

```java
@Nested
@DisplayName("when product is found")
class WhenProductIsFound {
    @Test
    @DisplayName("return HTTP status OK")
    void returnHttpStatusOk() throws Exception {
        Product product = new Product(1L, "Toothbrush", BigDecimal.valueOf(5.0));
        when(productRepository.findById(1L)).thenReturn(product);

        mockMvc.perform(get("/product/{productId}", 1L))
            .andExpect(status().isOk());
    }

    @Test
    @DisplayName("return found product as JSON")
    void returnFoundProductAsJson() throws Exception {
        Product product = new Product(1L, "Toothbrush", BigDecimal.valueOf(5.0));
        when(productRepository.findById(1L)).thenReturn(product);

        mockMvc.perform(get("/product/{productId}", 1L))
            .andExpect(jsonPath("$.id", is(1)))
            .andExpect(jsonPath("$.name", is("Toothbrush")))
            .andExpect(jsonPath("$.price", is(5.0)));
    }
}
```

由于在项目构造时存在重复代码，许多教程建议通过使用带有`@BeforeEach`注解的方法来消除重复：

```java
@Nested
@DisplayName("when product is found")
class WhenProductIsFound {
    @BeforeEach
    void productFound() {
        Product product = new Product(1L, "Toothbrush", BigDecimal.valueOf(5.0));
        when(productRepository.findById(1L)).thenReturn(product);
    }

    @Test
    @DisplayName("return HTTP status OK")
    void returnHttpStatusOk() throws Exception {
        mockMvc.perform(get("/product/{productId}", 1L))
            .andExpect(status().isOk());
    }

    @Test
    @DisplayName("return found product as JSON")
    void returnFoundProductAsJson() throws Exception {
        mockMvc.perform(get("/product/{productId}", 1L))
            .andExpect(jsonPath("$.id", is(1)))
            .andExpect(jsonPath("$.name", is("Toothbrush")))
            .andExpect(jsonPath("$.price", is(5.0)));
    }
}
```

不幸的是，该测试现在不是独立的，我们无法一眼看出与测试相关的所有内容，有更好的方法来消除重复，如辅助方法、测试数据构造器或对象母体。



进一步的检查示例代码，会发现第一个测试不关心属性值，而在第二个测试中字段值是关联信息。



现在可以看下使用构造器模式和辅助方法改写后的测试代码：

```java
@Nested
@DisplayName("when product is found")
class WhenProductIsFound {
    @Test
    @DisplayName("return HTTP status OK")
    void returnHttpStatusOk() throws Exception {
        havingPersisted(aProduct().withId(1L));

        mockMvc.perform(get("/product/{productId}", 1L))
            .andExpect(status().isOk());
    }

    @Test
    @DisplayName("return found product as JSON")
    void returnFoundProductAsJson() throws Exception {
        havingPersisted(aProduct().withId(1L).withName("Toothbrush").withPrice(5.0));

        mockMvc.perform(get("/product/{productId}", 1L))
            .andExpect(jsonPath("$.id", is(1)))
            .andExpect(jsonPath("$.name", is("Toothbrush")))
            .andExpect(jsonPath("$.price", is(5.0)));
    }
}
```

我们可以立即看到每个测试只有相关信息，同时还可以一眼看出测试方法的作用。



可读性和消除重复是一个宽泛的话题，我们不准备在此处详细讨论，对于可读性和可维护性的深入分析会放到其它文章中。

{{% admonition note "补充阅读" false %}}

* [**在代码中实践DRY和DAMP**](https://www.arhohuttunen.com/dry-damp-tests/)
* [**如何创建测试数据构造器**](https://www.arhohuttunen.com/test-data-builders/)
* [**如何让测试代码易于阅读**](https://www.arhohuttunen.com/test-readability/)

{{% /admonition %}}

## 总结

我们可通过嵌套测试来添加层级结构，可通过创建具有`@Nested`注解的内部类来创建嵌套测试。

嵌套测试有助于我们更好的描述测试方法之间的关系，嵌套测试还能够提高测试代码和测试结果的可读性和导航性。

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-nested-tests)中找到。

