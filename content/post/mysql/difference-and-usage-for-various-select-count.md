---
title: "MySQL中不同SELECT COUNT统计总数时的区别"
date: 2019-01-25T17:18:27+08:00
lastmod: 2019-01-25T17:18:27+08:00
draft: false
keywords: ["mysql","select count"]
description: "记录mysql中select count(id)、select count(*)以及select count(1)的区别"
tags: ["mysql"]
categories: ["数据库"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
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

在数据库中使用`COUNT`函数统计总数是常用操作，本文参考网上资料以及个人实际操作记录下`MySQL`中通过`COUNT(列名)`、`COUNT(常量)`以及`COUNT(*)`在**相同查询条件下[^1]** 的区别以及使用场景。

<!--more-->

本文基于[InnoDB](https://en.wikipedia.org/wiki/InnoDB)和[MyISAM](https://en.wikipedia.org/wiki/MyISAM)这两种常见的`MySQL`引擎，利用名为[add_user_batch](https://github.com/lucumt/myrepository/blob/master/mysql/add_user_batch.sql)的存储过程向`system_user`表中插入1000万数据，对比测试它们的查询结果和响应性能。

```mysql
CREATE TABLE `system_user` (
	`id` INT NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(8) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
	`age` INT DEFAULT NULL,
	`tag` VARCHAR(8) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
	PRIMARY KEY (`id`)
) ENGINE = INNODB CHARSET = utf8;
```

在`add_user_batch`中，采用循环的方式动态生成数据，每循环100次会将`insert`批量执行语句插入数据库中并提交，每循环25次其中的`tag`就会被置为空，故`tag`值为空的记录总共有400000个。

# 结果对比

在`MySQL`的官网中对于[COUNT()](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_count)函数有如下说明：

> Returns a count of the number of non-NULL values of expr in the rows retrieved by a `SELECT `statement. The result is a `BIGINT` value.
>
> If there are no matching rows,`COUNT()` returns 0. `COUNT(NULL)` returns 0.
>
> <br/>
>
> `COUNT(*)`is somewhat different in that it returns a count of the number of rows retrieved, whether or not they contain `NULL` values.

从中可以得出如下结论:

* `COUNT(expr)`计算的是`SELECT`操作获取的数据行中expr**不为空**的总数
* `COUNT(*)`计算的是`SELECT`操作获取的所有行数，不论其中是否有列为空

由于`COUNT()`主要是获取不为空的值，而在使用`COUNT(常量)`时其值恒为真，故其结果与`COUNT(*)`相同，同时由于主键不能为空，故`COUNT(主键)`的结果也与它们相同。基于上述理论分析可获知

> `COUNT(*)`=`COUNT(常量)`=`COUNT(主键)`>=[^2]`COUNT(非主键列)`

采用前面存储过程生成的数据分别用上述4种方式去查询的结果如下，由于tag值为空的记录数为400000个，故COUNT(tag)返回的记录数为9600000，实际查询结果均符合理论预期。

![innodb没有where条件查询](/blog_img/mysql/difference-and-usage-for-various-select-count/myisam-query-without-filter.png "innodb没有where条件查询") 

## 不同事务中的结果

由于`InnoDB`引擎支持事务，而在不同事务可能会导致数据库记录不一致，故在`MySQL`官网中对于`InnoDB`有如下文字说明，其主要说明的是`COUNT()`返回的是**当前事务中可见的对应行数** ，即同样的查询SQL在不同的事务中其结果可能不相同[^3]

> `InnoDB` does not keep an internal count of rows in a table because concurrent transactions might “see” different numbers of rows at the same time. Consequently, `SELECT COUNT(*)` statements only count rows visible to the current transaction.

# 性能对比

{{% admonition note "说明" false %}}

由于测试环境以及`MySQL`查询缓存的原因，即使是同一条`SQL`查询多次查询的时间消耗也不完全相同，故在性能对比这块只做大致时间的对比，不会精确到毫秒级。

{{% /admonition %}}

继续从`MySQL`官方中寻找相关说明:

> `InnoDB` handles `SELECT COUNT(*)` and `SELECT COUNT(1)` operations in the same way. There is no performance difference.
>
> <br>
>
> For `MyISAM` tables, [`COUNT(*)`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_count) is optimized to return very quickly if the [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html) retrieves from one table, no other columns are retrieved, and there is no `WHERE` clause.
>
> This optimization only applies to `MyISAM` tables, because an exact row count is stored for this storage engine and can be accessed very quickly. `COUNT(1)` is only subject to the same optimization if the first column is defined as `NOT NULL`

上述文字的要点如下：

* `InnoDB`对于`COUNT(*)`和`COUNT(1)`以相同的方式处理，它们没有性能上的区别
* `MyISAM`对于`COUNT(*)`在 **没有`WHERE`条件且只查询一张表** 的情况下会进行优化，而`COUNT(1)`只有在 **没有`WHERE`条件且第1列不为空** 的情况下才会进行优化

对于第1点可从前面结果对比查询图中得到验证，其查询时间都近似为0.01s。



下面分别展示`MyISAM`在有和没有`WHERE`过滤条件时`COUNT()`函数的查询耗时：

* 没有`WHERE`条件执行如下，从中可以看出只有对于可能存在空值的tag列，其查询耗时为2.26s，其余的耗时均为0.01s，这其中的特例是name列，虽然不是主键，但是由于在建表时限制其非空，故`InnoDB`引擎会对其进行优化处理。

  ![myisam没有where条件查询](/blog_img/mysql/difference-and-usage-for-various-select-count/myisam-query-without-filter.png "myisam没有where条件查询") 

* 有`WHERE`条件执行如下，从图中可以看出，此时由于`InnoDB`引擎优化不生效，故它们的查询时间都在秒级范围。

  ![myisam有where条件查询](/blog_img/mysql/difference-and-usage-for-various-select-count/myisam-query-with-filter.png "myisam有where条件查询") 

* 给system_user表添加名为type的列并放在第1列，然后分别执行`COUNT(1)`与`COUNT(*)`发现耗时近似相，官方文档上说的**只有在第1列不为空**的限制条件在此处并不生效，原因待进一步分析。

  ![myisam第1列为空查询](/blog_img/mysql/difference-and-usage-for-various-select-count/myisam-query-first-column-is-null.png "myisam第1列为空查询") 



# 总结&建议

* 总结:
  * `SELECT COUNT(*)`，查询特定表总行数时
  * `SELECT COUNT(1)`，查询特定表总行数，其结果同`SELECT COUNT(*)`
  * `SELECT COUNT(列名)`,查询指定列中符合条件得所有非空值
* 使用建议:
  * `SELECT COUNT(*)`，查询总行数时使用，尤其是`MyISAM`引擎会在特定场景下进行优化
  * `SELECT COUNT(1)`，由于在`MyISAM`中只有在特定场景下优化才会生效，此种用法较为偏僻不符合SQL规范，不建议使用
  * `SELECT COUNT(列名)`，查询对应列的非空总行数

参考文档:

1. https://segmentfault.com/a/1190000040733649
2. https://stackoverflow.com/questions/2710621/count-vs-count1-vs-countpk-which-is-better
3. https://github.com/lucumt/myrepository/blob/master/mysql/add_user_batch.sql

[^1]: 即`WHERE`后面的过滤条件相同
[^2]: 相等的场景为`SELECT`返回的行中该列数据全部不为空
[^3]: 依赖于具体的事务隔离级别