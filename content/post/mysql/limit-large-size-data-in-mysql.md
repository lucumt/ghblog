---
title: "在MySQL中对大量数据进行limit offset分页查询的优化"
date: 2019-02-11T09:52:31+08:00
lastmod: 2019-02-11T09:52:31+08:00
draft: false
keywords: ["mysql","limit"]
description: "在MySQL中对大量数据进行limit offset分页查询的优化"
tags: ["mysql"]
categories: ["数据库"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

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

---

在软件开发中采用`LIMIT OFFSET`对数据库进行分页是常见操作，但在数据量很大时直接使用`LIMIT OFFSET`查询尾部的数据会导致性能很慢，本文简要介绍2种改进方案。

<!--more-->

以`system_user`表为例，基于[MySQL中快速创建大量测试数据](../mysql-create-massive-test-data-quickly/)一文中的介绍给其添加1000万的测试数据

```sql
CREATE TABLE `system_user` (
	`id` INT NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(8) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
	`age` INT DEFAULT NULL,
	`tag` VARCHAR(8) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
	PRIMARY KEY (`id`)
) ENGINE = INNODB CHARSET = utf8;
```

基于查询SQL`SELECT * FROM system_user LIMIT 9999990,10;`进行优化分析。

上述SQL在数据库中的查询耗时如下：

![原始查询耗时](/blog_img/mysql/limit-large-size-data-in-mysql/mysql-origional-limit-offset-query.png "原始查询耗时") 

利用`explain`分析其执行计划结果如下：

![原始查询耗时分析](/blog_img/mysql/limit-large-size-data-in-mysql/mysql-origional-limit-offset-query-explain.png "原始查询耗时分析") 

上图中的`rows`这一例的数据为9722930，由于`rows`这一列的值是预估值，实际上`MySQL`会将数据`offset+count`的数据都获取到内存中，然后再进行过滤。在本例中即会将10000000条数据都获取到然后再进行过滤筛选，而获取这么多数据显然会导致查询速度变慢！

> **问题根源为`MySQL`在执行`LIMIT OFFSET`时会将数据全部加载到内存中然后再进行过滤，实际上执行的是一种假分页！**

找到问题的根源后，要提高查询速度只能让`MySQL`查询时返回的数据尽可能小，接下来根据主键是否连续自增来分别叙述。

# 自增主键过滤

若**主键连续自增**，则可从业务逻辑的角度先对数据用`WHERE`过滤，然后用`LIMIT`进行分页，类似SQL如下：

```sql
SELECT * FROM `system_user` WHERE id>=9999990 LIMIT 10;
```

执行结果如下，可以看出时间明显缩短很多

![自增主键过滤查询](/blog_img/mysql/limit-large-size-data-in-mysql/mysql-auto-id-filter-limit-offset-query.png "自增主键过滤查询") 

进一步分析其执行计划，发现`rows`这一列的值为11，只是获取了我们想要的数据，没有获取大批量数据。

![自增主键过滤查询分析](/blog_img/mysql/limit-large-size-data-in-mysql/mysql-auto-id-filter-limit-offset-query-explain.png "自增主键过滤查询分析") 

采用此种方式性能提升的原因如下：

1. `MySQL`中的主键默认有索引，基于索引查询速度很快
2. `WHERE`优先于`LIMIT`执行，数据量相对之前，变得很小

其中最关键的是第2点，只要查询的数据量变小，查询速度自然会提升。 

# 覆盖索引过滤

基于自增主键过滤要求主键必须**主键连续自增**，若主键不连续(如主键采用`UUID`生成)则上述方案不可行，此时可基于[覆盖索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_covering_index)来减少获取和传输的数据量大小。

将查询sql修改为类似如下

```sql
SELECT u1.* FROM `system_user` u1
JOIN 
(SELECT id FROM `system_user` LIMIT 9999990,10) u2 ON u1.id=u2.id;
```

执行结果如下，可以看出耗时只比最初的少1秒

![自增主键limit查询](/blog_img/mysql/limit-large-size-data-in-mysql/mysql-join-filter-limit-offset-query.png "自增主键limit查询") 

查看其执行计划，发现获取的数据量仍然很大

![自增主键limit查询分析](/blog_img/mysql/limit-large-size-data-in-mysql/mysql-join-filter-limit-offset-query-explain.png "自增主键limit查询分析") 

由于`MySQL`在数据量为千万级时查询速度会变慢，将数据库表中的数据量缩小到500万，执行如下：

![自增主键limit缩小数据查询](/blog_img/mysql/limit-large-size-data-in-mysql/mysql-join-filter-limit-offset-query-small.png "自增主键limit缩小数据查询") 

问题的根源在`SELECT id FROM system_user LIMIT 9999990,10`，虽然此时只查询id，但是id的数量仍然很庞大，由此造成查询速度变慢。



此时可通过在数据库表中添加一列`num`并对其创建唯一索引，之后基于`num`进行过滤查询

```sql
-- 添加索引
ALTER TABLE `system_user` ADD COLUMN `num` INT NOT NULL DEFAULT 1;
ALTER TABLE `system_user` ADD UNIQUE INDEX `user_num_index` (`num`);

-- 重新制造数据
TRUNCATE TABLE `system_user`;
CALL add_user_batch(10000000);
```

改进后的sql如下

```sql
SELECT u1.* FROM `system_user` u1
JOIN 
(SELECT num FROM `system_user` WHERE num>=9999990 LIMIT 10) u2 ON u1.num=u2.num;
```

执行结果耗时如下：

![利用索引列过滤查询](/blog_img/mysql/limit-large-size-data-in-mysql/mysql-unique-index-filter-limit-offset-query.png "利用索引列过滤查询") 

可以看出其耗时和采用**自增连续主键**时类似。对应的执行计划如下，从图中也能看出要获取的数据量明显变小。

![利用索引列过滤查询分析](/blog_img/mysql/limit-large-size-data-in-mysql/mysql-unique-index-filter-limit-offset-query-explain.png "利用索引列过滤查询分析") 

# 总结

上述两种方案归根到底均为要通过`WHERE`提前过滤不需要的数据，减少返回的数据量，总结如下：

* 若**主键连续且自增**，则通过主键进行过滤
* 若**主键不连续自增**，可额外创建一个自增列或者采用`覆盖索引`的方式改写