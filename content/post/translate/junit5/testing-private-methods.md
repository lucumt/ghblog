---
title: "[译]如何测试私有方法"
date: 2023-07-03T22:42:07+08:00
lastmod: 2023-07-03T22:42:07+08:00
draft: false
keywords: ["junit","private"]
description: "JUnit翻译专题，主要是如何在JUnit中测试私有方法"
tags: ["java","junit"]
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

本文翻译自[**How Do I Test Private Methods?**](https://www.arhohuttunen.com/testing-private-methods/)

<!--more-->

![Man With Laptop](/blog_img/translate/junit5/testing-private-methods/man-using-laptop.webp "Man With Laptop") 

> 测试私有方法的最佳方式是什么，使用一些库还是使用反射？

不要直接测试私有方法，测试私有方法的最佳方法是通过另一个公共方法，私有方法是公共方法的实现细节。

> 但是，但是……通过公共方法来测试私有方法可能会很困难。

如果是这种情况，那么意味着测试的类设计不太好，如果测试设置太复杂，该类可能会尝试做太多事情，此时应该看看是否可以在新类中提取出一些功能。

> 但是，但是……如果添加更多的类，不是会增加复杂性吗？

在大多数情况下，添加新类会降低而不是增加复杂性，当一个类有一个明确的、小的职责时，它就很容易理解和测试。

> 但是，但是……所以仅仅只是为了测试我们需要编写10个小类来替代私有方法？

我并不是建议你应该创建一个新类只是为了测试方法，我建议如果你关注输出则可以改进你的设计，诸如复杂的测试设置之类的事情是关于实现复杂性的提示线索。

> 但是，但是……能否把这个方法公开？

仅仅为了测试而暴露方法并不是一个好主意，它会破坏封装并泄漏内部结构且每个人都能访问，这可能会产生有害的副作用，测试私有方法的设计可能意味着它是属于另一个类的单独职责的一部分。

> 但是，但是……如果不能将方法公开，那么提取到一个单独的类不也是做同样的事情吗？

如果将某些内容移动到另一个类让你感到困惑，那么除了`public`和`private`之外还有更多的可见性修饰符，如`package-private`和`protected`，可利用这些优势来发挥，如果测试类与目标类位于同一个包中，则没有问题。

> 但是，但是……我想单独测试代码，单元测试不是应该被隔离吗？

隔离通常意味着与服务、数据库或文件系统等外部依赖项的隔离。

如果直接测试私有方法，则每次重构类时测试都会中断，另一方面如果只测试公共方法，则可以在不破坏测试的情况下重构类内部。

> 但是，但是……测试和代码不是应该齐头并进吗？

是的，但仅限于公共协作层面，如果直接测试实现逻辑而有人想对其进行小更改，那么他也必须更改测试。

如果重复这样做，单元测试就会开始阻碍开发并成为开发团队的烦恼，如果不能在不改变测试的情况下改变你的实现，那么测试策略就不合适。

> 但是，但是……如果我喜欢采用自下而上的方法，并且在编写私有方法时没有公共方法怎么办？

也许你应该尝试自上而下的方法，如果先编写私有方法，你就会猜测你需要什么，如果最后对解决方案不满意则必须重写所有测试，从公共方法开始编写会让你思考它将如何使用。

> 但是，但是……如果我开发出解决方案，但测试没有通过，那不是很难知道问题出在哪里吗？

不完全这样，你不会一次性实施所有事情，你将从一个简单的测试和一个简单的解决方案开始，之后可以逐步添加测试和功能，可以获得相同的覆盖范围，并且可以在不破坏测试的情况下重构实现。

> 嗯... 好吧，我还没有100%相信，但你已经提出了一些很好的观点，让我考虑一下。

好极了，稍后告诉我你的想法。