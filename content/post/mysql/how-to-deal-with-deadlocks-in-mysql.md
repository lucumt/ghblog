---
title: "[译]如何处理MySQL中的死锁"
date: 2021-09-25T17:03:21+08:00
lastmod: 2021-09-25T17:03:21+08:00
draft: false
keywords: ["mysql","死锁"]
description: "翻译外网文章，简要介绍如何处理MySQL中的死锁问题，以便开发中遇到类似问题时可以参考"
tags: ["mysql"]
categories: ["数据库","翻译"]
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

本文翻译自[**How to deal with deadlocks in mysql**](https://medium.com/geekculture/how-to-deal-with-deadlocks-in-mysql-58f4d830788b)

<!--more-->

尽管对某些人来说很神秘，死锁却是一个很常见的问题。大多数时候由于事务将会回滚，死锁是没有副作用的，但若处理不当则可能会变得很糟糕。

## 定义

![死锁定义](/blog_img/mysql/how-to-deal-with-deadlocks-in-mysql/dead-lock-definition.webp "死锁定义") 

死锁是多个事务(通常为2个)在互相等待彼此的锁，通常`MySQL`可通过回滚事务来自行检测并解决此类问题，除非关闭死锁检测功能。

死锁不仅仅存在于数据库中，只要有并发场景，就可能产生死锁！

## 处理策略

处理死锁的策略包括3个步骤：

1. 识别有问题的事务
2. 源码中定位到相关事务
3. 找到合适的解决方案

### 识别有问题的事务

首先需要获取死锁相关的信息，可通过`SHOW ENGINE INNODB STATUS`这条简单的的指令实现，不过其只显示最新的死锁信息，这对于了解一段时间内的死锁全过程信息没有太多帮助。

幸好，从`MySQL5.5.30`开始其提供了一个名为`innodb_print_all_deadlocks`的配置项用于将所有的死锁都打印到错误日志中，同时启用此功能也不会导致停机。



相关的日志内容看起来类似如下

```sql
*** (1) TRANSACTION:
TRANSACTION 450913541522, ACTIVE 0 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 7 lock struct(s), heap size 1136, 3 row lock(s), undo log entries 2
MySQL thread id 93608210, OS thread handle 47321957033728, query id 10802490531 10.0.64.165 db updating
update `table1` set `scroll_to` = 3901, `updated_at` = ‘2021–08–17 11:30:53’ where `id` = 668126442
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 1455 page no 5719543 n bits 0 index PRIMARY of table `db`.`table1` trx id 450913541522 lock_mode X locks rec but not gap waiting
Record lock, heap no 95 PHYSICAL RECORD: n_fields 24; compact format; info bits 0
*** (2) TRANSACTION:
TRANSACTION 450913541519, ACTIVE 0 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 9 lock struct(s), heap size 1136, 4 row lock(s), undo log entries 4
MySQL thread id 93608214, OS thread handle 47339872995072, query id 10802490581 10.0.63.235 admin update
insert into `table2` (`user_id`, `company_id`, `date`, `steps`, `updated_at`, `created_at`) values (491031, 1, ‘2021–08–17’, 359, ‘2021–08–17 11:30:53’, ‘2021–08–17 11:30:53’)
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 1455 page no 5719543 n bits 0 index PRIMARY of table `db`.`table1` trx id 450913541519 lock_mode X locks rec but not gap
Record lock, heap no 95 PHYSICAL RECORD: n_fields 24; compact format; info bits 0
*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 5817 page no 4486 n bits 0 index PRIMARY of table `db`.`user` trx id 450913541519 lock mode S locks rec but not gap waiting
Record lock, heap no 99 PHYSICAL RECORD: n_fields 29; compact format; info bits 0
```

尽管其中包含许多信息，但由于我只关心哪个事务/线程在持有/等待哪些锁，将上述内容精简如下：

```sql
Transaction 1: update `table1` set `scroll_to` = 3901, `updated_at` = ‘2021–08–17 11:30:53’ where `id` = 668126442
-> Holding an X lock on `db`.`user`
-> Wait for an X lock on `db`.`table1`
Transaction 2: insert into `table2` (`user_id`, `company_id`, `date`, `steps`, `updated_at`, `created_at`) values (491031, 1, ‘2021–08–17’, 359, ‘2021–08–17 11:30:53’, ‘2021–08–17 11:30:53’)
-> Hold an X lock on `db`.`table1`
-> Wait for an S lock on `db`.`user`
```

可以看出尽管日志中没有明确的说明事务1持有了哪个锁，我们仍然可以推断出事务1在`db.user`中持有锁，而事务2正在等待该锁。



同样可看出事务1虽然只是更新`table1`，但却在表`user`上持有锁，对事务2也类似，其向`table2`中插入数据但却在`table1`上持有锁，然后等待`user`表中的锁。咋一看很奇怪，这是由于此处显示的查询语句只是整个事务的**一部分**，这意味着在更新表`table1`之前，事务1已经在`user`表上执行了某些操作因而持有X锁，此分析同样适用于事务2，这些持有/获取锁的信息有助于在源码中具体定位事务的位置。

### 源码中定位到相关事务

接下来就是在应用程序源码中找到事务，通常是通过编辑器的搜索功能实现，事务中应该包含我们在步骤1中发现的持有/等待锁相关的查询。



对于大型代码工程来说可能是一项艰巨的任务，尤其是哪些使用了ORM框架实现数据库交互的项目。虽然数据库表名有助于缩小查询范围，但大多数情况下更需要耐心，或许同样需要检查由死锁导致的应用程序日志，其中包含了代码行等信息。

### 找到合适的解决方案

深入研究应用程序代码之后，假设你已经找到问题产生的根源，接下来需要寻找到一个合适的解决方案。



并发是死锁产生的根源，为了减少死锁发生的概率，事务应该尽可能的快以避免受到其它事务的干扰，可通过下述方式重构来加快事务执行：

* 将业务代码放到事务之外，我在遗留项目中看到过充斥着业务逻辑的臃肿事务。
* 优化慢查询



另一种方法是通过同步调用来避免并发。例如在打开一个应用程序时，用户位置和用户活动信息是以异步方式(同时的)发送到服务器，从而导致数据局死锁。将这些调用转化为同步实现应该能消除死锁，因为没有并发就没有死锁，当无法对遗留项目进行重构时该方案是一个不错的选择，然而若业务逻辑不允许这样修改则不生效。

## 总结

就像人们所说的那样，死锁神秘且令人烦恼，但如果我们知道在哪寻找且具有一些耐心，其就不会难么神秘和令人讨厌且肯定能否避免。