---
title: "[译]Junit 5 预期异常 - 如何检测异常是否抛出"
date: 2023-07-02T14:21:43+08:00
lastmod: 2023-07-02T14:21:43+08:00
draft: false
keywords: ["junit","exception"]
description: "JUnit5翻译专题，主要是Junit 5 预期异常以及如何检测异常是否抛出"
tags: ["java","junit","junit5"]
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

本文翻译自[**JUnit 5 Expected Exception: How to assert an exception is thrown?**](https://www.arhohuttunen.com/junit-5-expected-exception/)

<!--more-->

![JUnit 5 Service Error](/blog_img/translate/junit5/junit-5-expected-exception/junit-5-laptop-with-service-error.webp "JUnit 5 Service Error") 

在本文中，我们将学习如何使用`JUnit 5`断言抛出异常以及如何检查抛出异常的错误消息。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 概览

为了确保我们的错误处理逻辑正常工作，验证一段代码在某些条件下是否出发特定异常是有必要的。

## 断言抛出异常

断言一段代码抛出特定的异常可通过使用`JUnit 5`中的`assertThrows()`方法来完成：

```java
@Test
void notEnoughFunds() {
    BankAccount account = new BankAccount(9);
    assertThrows(NotEnoughFundsException.class, () -> account.withdraw(10),
            "Balance must be greater than amount of withdrawal");
}
```

在此示例中，如果我们尝试从帐户余额允许的银行帐户中提取更多资金，则相关代码实现会抛出`NotEnoughFundsException`异常。

## 失败的断言

现在，假设在我们的示例中忘记在提款前检查余额，如果不抛出异常，测试将失败并显示错误消息：

```bash
Balance must be greater than amount of withdrawal ==> Expected com.arhohuttunen.junit5.exception.NotEnoughFundsException to be thrown, but nothing was thrown.
org.opentest4j.AssertionFailedError: Balance must be greater than amount of withdrawal ==> Expected com.arhohuttunen.junit5.exception.NotEnoughFundsException to be thrown, but nothing was thrown.
```

此外，假设在我们的示例中我们忘记初始化余额，代码将抛出`NullPointerException`异常，如果抛出非预期异常，测试将失败并显示不同的错误消息：

```bash
Balance must be greater than amount of withdrawal ==> Unexpected exception type thrown ==> expected: <com.arhohuttunen.junit5.exception.NotEnoughFundsException> but was: <java.lang.NullPointerException>
org.opentest4j.AssertionFailedError: Balance must be greater than amount of withdrawal ==> Unexpected exception type thrown ==> expected: <com.arhohuttunen.junit5.exception.NotEnoughFundsException> but was: <java.lang.NullPointerException>
```

## 断言异常消息

此外，有时我们想要验证异常的一些信息，例如错误消息或原因，在这种情况下可以捕获抛出的异常：

```java
@Test
void notEnoughFundsWithMessage() {
    BankAccount account = new BankAccount(0);
    Throwable thrown = assertThrows(NotEnoughFundsException.class, () -> account.withdraw(100));
    assertEquals("Attempted to withdraw 100 with a balance of 0", thrown.getMessage());
}
```

## 总结

`JUnit 5`可以轻松断言使用`assertThrows()`方法引出发预期的异常，此外也可以捕获抛出的异常以检查错误消息等更多信息。

本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-expected-exception)中找到。
