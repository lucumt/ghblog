+++
author = "飞狐"
categories = ["编程杂想"]
date = "2017-12-23T12:07:40+08:00"
description = "从复杂代码中找出Bug的几种方法"
keywords = ["Coding"]
title = "从复杂代码中找出Bug的几种方法"

+++

工作中有时候会遇到某些大段复杂代码出现Bug的情况，不同于一般行数较小或逻辑较简单的代码，对于大段复杂的代码进行分析可能会很耗时，本文介绍几种个人在工作中用到的方法，供大家参考。

<!--more-->

## 将Log级别开启到Debug

将Log的级别变为Debug后，会比INFO状态下看到更多的详细日志信息，仔细分析这些输出信息，有时候可以发现是哪一步不能按照预期工作，据此缩小查找范围，从而解决缺陷。  
!["Debug级别的日志输出"](/blog_img/other/different-ways-find-bug-in-complex-code/debug_log_output.png "Debug级别的日志输出")   
实际使用中由于Debug级别的log输出信息庞大，查看输出log很耗时，在实际开发中用得不多，常用的场景如下：

- 知道大致的代码范围，通过观察log输出来确定具体的原因
- 压根不知道问题出在哪里，通过观察log输出来获取有用信息，辅助以其它手段来分析
- 多线程等无法利用Debug调试的环境
  

## 利用IDE进行Debug

在IDE中开启Debug模式进行调试是开发过程中最常用的一种手段，通过Debug调试可以找出哪些代码被执行，哪些没有被执行，以及在执行过程中相关变量的值。Debug调试在大部分情况下都可以帮我们找出缺陷，但在多线程应用、 HTML页面样式调试等场景下并不适用Debug方式。  
!["在IDE中进行Debug"](/blog_img/other/different-ways-find-bug-in-complex-code/debug_in_ide.png "在IDE中进行Debug")

## 在代码仓库中进行版本回溯

若已知出问题的代码在以前没有问题，只是最近的更改才出现问题，可以通过从代码仓库中找出历史版本与目前的版本进行对比，看看有哪些差异，通常问题都发生在这些有差异的地方。


与其它方式相比此种方法耗时最少，通过这种方式找出的bug一般都是某些关键配置写错了或代码语法不正确时控制台没有输出完整的错误信息，如在Ajax方法中在某些配置项后面少写了一个 `,`时，在某些浏览器中进行调试时只会出现 `Uncaught SyntaxError: Unexpected identifier` 这种错误消息，无法根据错误信息具体定位到哪一行代码出错。

## 通过二分查找定位

我最喜欢用的方法之一，二分查找定位的操作和二分查找算法类似，先将一部代码注意掉或插入试探性代码，确定问题是在这一边或那一边，确定完大致的区域后，对该问题区域再次采用二分查找来定位，直到找到问题原因为止。此种方法和二分查找算法一样耗时较少。
!["通过二分查找确认问题"](/blog_img/other/different-ways-find-bug-in-complex-code/binary_search_debug.png "通过二分查找确认问题")     
此方法常用的场景如下：

- 不方便利用Debug日志或Debug调试的地方
- HTML 、JSP等View层的代码
- 刚上手某项技术，对其原理了解不深入


## 重新编写代码实现

若前面几种方法均不能凑效，可以采取终极大法： **重新编码实现！** ，`不识庐山真面目，只缘身在此山中` ，有些代码的实现逻辑本身就有问题，若直接对其进行Debug调试或分析，可能会陷入该代码错误的陷阱中。如我们可能认为某段代码代码是没有问题的，然后基于该代码进行进一步的分析调试，却无论如何都得不到自己想要的结果，问题的原因就在于该段代码本身就是错误的。起始点选错了，无论后面怎么做，都无济于事。

根据原始的需求，在不参照现有代码实现方式的基础上，以白板的方式重新编程实现功能，在对比旧的代码，就能发现问题产生的原因。若这种方式还不能凑效，可以让其他同事协助审查自己的代码，或者审核原有的需求本身是否有问题，通常这种情形意味着我们的代码必须要重构。

## 总结
好的代码会自己说话，平时编码时要注意细节、符合规范，必要时对代码进行重构，将复杂的代码进行抽取简化。代码重构虽然对我们的工作产出不会有立竿见影的效果，但也绝不是白白浪费我们的时间，不断重构后的实现良好的代码，不仅有利于调试分析，也有利于他人顺利接手。

在一些成立时间较长的公司或多或少都会存在一些**炸弹代码**，此处说的炸弹代码是指那些功能很重要实现又很复杂的代码，最开始开发该功能的人离职后，后面接手的人看不懂该代码，由于各种原因无法对其进行重构，在后续开发时，只能不断的在其后面添加新的业务逻辑，而不敢修改已有的，导致代码量越来越臃肿，越臃肿就越看不懂，陷入恶性循环。这样的代码在后续每次调试分析时都是大坑，而如果一开始就进行适当的重构，或许不会发生这种情况。

Orz~ 我承认**炸弹代码** 这个词是我自己发明的！
