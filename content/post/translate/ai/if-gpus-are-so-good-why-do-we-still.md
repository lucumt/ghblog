---
title: "[译]既然GPU如此强大，为啥我们还需要使用CPU?"
date: 2025-02-17T11:21:37+08:00
lastmod: 2025-02-17T11:21:37+08:00
draft: false
keywords: ["CPU","GPU"]
description: ""
tags: ["AI"]
categories: ["翻译","人工智能"]
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
codeTabSeperator: "::"
moreMeta: true

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

本文翻译自[**If GPUs Are So Good, Why Do We Still Use CPUs At All?**](https://codingstuff.substack.com/p/if-gpus-are-so-good-why-do-we-still)

<!--more-->

最近Twitter上出现了一段2009年的老视频，它旨在让观众更直观地了解CPU和GPU之间的区别。

可观看下述时长90秒的视频以了解更多:

{{< youtube -P28LKWTzrI >}}

其主要让CPU和GPU进行一场面对面的绘画比赛，相关处理器连接到发射彩蛋的机器上。

CPU需要整整30秒才能绘制一个非常基本的笑脸：

![CPU绘画结果](/blog_img/translate/ai/if-gpus-are-so-good-why-do-we-still/cpu-paint-result.webp "CPU绘画结果") 

而GPU却在瞬间画出了蒙娜丽莎：

![GPU绘画结果](/blog_img/translate/ai/if-gpus-are-so-good-why-do-we-still/gpu-paint-result.webp "GPU绘画结果") 

这段视频和核心观点是CPU速度慢而GPU速度快，虽然是事实，但其中还有很多细节没有透露。

## 每秒万亿次浮点运算

当我们谈论GPU比CPU的性能更好时，其实指的是一个指标TFLOPS，它本质上衡量的是处理器在一秒钟内可以进行多少万亿次数学运算。例如Nvidia A100 GPU每秒可进行9.7万亿次运算，而最近的Intel 24核心处理器每秒只能执行0.33万亿次运算，这意味着中等水平的GPU的运算速度至少比功能最强大的CPU快30倍。



在我个人的MacBook中(Apple M3芯片)同时包含了GPU和CPU，能否去掉这些运行缓慢的CPU呢？

## 不同类型的程序

接下来定义两种类型的程序来说明此问题：`串行程序`和`并行程序`。

### 串行程序

`串行程序`指的是其中的所有指令都依次按序执行，下述代码即是一个例子：

```go
def sequential_calculation():
    a = 0
    b = 1

    for _ in range(100):
        a, b = b, a + b
        return b
```

在此段代码中，我们连续100次使用前2个数字来计算下一个数字，其重要特性是**每个步骤都依赖于之前的2个步骤**。如果你想通过手工计算来获取结果，则不能对你朋友说“让他从51步算到100步，而自己则从第1步到50步”，这是由于要计算第51的结果时需要获取49和50步的结果，每1个步骤的值都依赖前2个步骤的值来计算。

### 并行程序

并行程序则指的是指令结果不彼此依赖，可互相执行，下述代码为一个示例：

```python
def parallel_multiply():
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    results = []

    for n in numbers:
        results.append(n * 2)

    return results
```

在这个例子中，我们执行10次互相独立的乘法运算，核心是**它们的执行顺序对最终结果无影响**，如果你想将部分任务分给你朋友，则可以说“你执行奇数乘法，我执行偶数乘法”，我们可以分别并行的运算并得到一个正确结果。

## 错误的二分法

事实上前面的划分是一种错误的二分法(关于`并行程序`的说明)，现实世界中的大部分大型程序混合着串行和并行代码，实际上几乎所有程序都有一定比例的指令/代码可以并行化。



假设我们有个如下所示的程序需要执行20次计算，前10次是计算斐波那契数，它们需要依次计算，而之后的10次计算则可以并行计算，由于其中一半的指令可以独立并行执行，则可说该段代码可50%并行化。

```python
def half_parallelizeable():
    # Part 1: 串行计算
    a, b = 0, 1
    fibonacci_list = [a, b]
    for _ in range(8):  # 计算另外8个数
        a, b = b, a + b
        fibonacci_list.append(b)

    # Part 2: 每个步骤都是独立的
    parallel_results = []
    for n in fibonacci_list:
        parallel_results.append(n * 2)
    
    return fibonacci_list, parallel_results
```

由于每个斐波那契数依赖于前面的两个数，所以前半部分是串行化，而后半部分则可采用完整的列表将每个数字单独翻倍并行计算。



我们无法在不计算第6个和第7个斐波那契数的前提下计算第8个斐波那契数，但是一旦获取了完整的斐波那契数，则可将翻倍计算分发到任意多个可用的执行器上执行。

## 不同的处理器处理不同的程序

通常来说，CPU擅长`串行程序`而GPU则更擅长`并行程序`，这是因为CPU和GPU存在根本的设计差异。



CPU具有少量的大内核(Apple的M3有8和CPU)，GPU则拥有大量的小内核（Nvidia的H100 GPU拥有上千个内核），这是GPU善于执行高度并行的程序的原因，它具有上千个简单的内核可用于对不同片段的数据并行的执行相同的操作。



渲染视频游戏图形就是一种需要进行许多简单重复计算的应用程序。想象下我们的游戏屏幕是一个巨大的像素矩阵，当你突然将角色转向右侧时，这些像素都需要重新计算新的颜色值，幸运的是，屏幕顶部的像素计算与底部的像素计算是互相独立的，因此可将计算任务交给上千个GPU内核。这也是GPU对于游戏如此重要的原因。

## CPU擅长处理随机事件

在执行高度并行的任务（例如将 10,000个独立数字的矩阵相乘）时，CPU的速度比GPU慢得多，但CPU擅长复杂的顺序处理和复杂的决策。



将CPU内核想象成一个繁忙的厨师长，他能够：

* 当VIP客人有特殊饮食要求时，立即调整烹饪计划
* 在准备精致酱汁和检查烤蔬菜之间无缝切换
* 通过重新组织整个厨房工作流程来处理停电等意外情况
* 精心安排多道菜品，让它们在恰当的时机送达，既热又新鲜
* 在处理数十个处于不同完成状态的订单的同时保持食品质量

相比之下，GPU内核就像一百名擅长重复性任务的一线厨师，他们可以在两秒钟内切好1个洋葱，但无法有效地管理整个厨房，如果要求GPU处理需求不断变化的晚餐服务，它将会很吃力。



这就是为什么CPU对于运行计算机操作系统至关重要，现代计算机面临着源源不断的不可预测事件，app的启动与停止、网络掉线、文件访问、用户随机点击屏幕等。CPU擅长处理所有这些任务，同时保持系统响应能力，它可以立即从帮助 Chrome呈现网页切换到处理Zoom视频通话，再到处理新的USB设备连接——同时跟踪系统资源并确保每个应用程序都得到公平的关注。



因此，尽管GPU擅长并行处理，但CPU仍然因其处理复杂逻辑和适应不断变化的条件的独特能力而不可或缺，Apple的M3等现代芯片兼具两者，将CPU灵活性与GPU计算能力相结合。



事实上，更精确的绘画视频会显示CPU管理图像下载和内存分配，然后再调用GPU快速渲染像素。