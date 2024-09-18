---
title: "在基于MySQL的Grafana的柱状中不显示0值"
date: 2024-04-19T17:21:43+08:00
lastmod: 2023-04-19T17:21:43+08:00
draft: false
keywords: ["grafana","mysql","deveops","柱状图"]
description: "简要记录如何在基于MySQL的Grafana的柱状中不显示0值"
tags: ["devops","grafana","mysql"]
categories: ["工具使用"]
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
centerImage: false
borderImage: true

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

简单记录下自己在使用`Grafana`过程中的一点点经验总结：通过修改sql的方式来消除柱状图中多余的0值。

<!--more-->

## 问题

通过`Grafana`基于`MySQL`查询系统监控统计数据时，发现在柱状图中有很多0值，严重影响使用体验。

![柱状图显示0值](/blog_img/devops/hidden-zero-value-in-grafana-bar-chart/bar-chart-with-zero-values.png  "柱状图显示0值")

## 解决

一开始自己想通过`Grafana`面板的过滤这些0值，搜索一番后并没有找到基于`Grafana`的解决方案，给出的答案是通过修改sql来规避此问题[^1]，对比该答案，还发现自己遇到的问题类似。

原始查询sql：

```sql
SELECT 
    DATE_FORMAT(t.start_time,'%Y-%m-%d') AS `taskDate`,
    SUM(IF(t.instance_status = 0,1,0)) AS `未知`,
    SUM(IF(t.instance_status = 1,1,0)) AS `等待触发`,
    SUM(IF(t.instance_status = 2,1,0)) AS `队列中`,
    SUM(IF(t.instance_status = 3,1,0)) AS `开始执行`,
    SUM(IF(t.instance_status = 4,1,0)) AS `执行中`,
    SUM(IF(t.instance_status = 5,1,0)) AS `成功`,
    SUM(IF(t.instance_status = 6,1,0)) AS `失败`,
    SUM(IF(t.instance_status = 7,1,0)) AS `终止`,
    SUM(IF(t.instance_status = 8,1,0)) AS `超时`,
    SUM(IF(t.instance_status = 9,1,0)) AS `拒绝执行`,
    SUM(IF(t.instance_status = 10,1,0)) AS `已访问`
    FROM t_work_flow_instance t 
WHERE t.start_time> DATE_SUB(NOW(), INTERVAL 14 DAY)
  GROUP BY taskDate
  ORDER BY taskDate
```

`mysql`中查询结果：

![mysql原始查询结果](/blog_img/devops/hidden-zero-value-in-grafana-bar-chart/mysql-query-result-1.png  "mysql原始查询结果")

**为了解决该问题，需要将`IF`中的默认值从`0`修改为`null`。**

改进后的sql:

```sql
SELECT 
    DATE_FORMAT(t.start_time,'%Y-%m-%d') AS `taskDate`,
    SUM(IF(t.instance_status = 0,1,null)) AS `未知`,
    SUM(IF(t.instance_status = 1,1,null)) AS `等待触发`,
    SUM(IF(t.instance_status = 2,1,null)) AS `队列中`,
    SUM(IF(t.instance_status = 3,1,null)) AS `开始执行`,
    SUM(IF(t.instance_status = 4,1,null)) AS `执行中`,
    SUM(IF(t.instance_status = 5,1,null)) AS `成功`,
    SUM(IF(t.instance_status = 6,1,null)) AS `失败`,
    SUM(IF(t.instance_status = 7,1,null)) AS `终止`,
    SUM(IF(t.instance_status = 8,1,null)) AS `超时`,
    SUM(IF(t.instance_status = 9,1,null)) AS `拒绝执行`,
    SUM(IF(t.instance_status = 10,1,null)) AS `已访问`
    FROM t_work_flow_instance t 
WHERE t.start_time> DATE_SUB(NOW(), INTERVAL 14 DAY)
  GROUP BY taskDate
  ORDER BY taskDate
```

`mysql`中查询结果如下，可看出除了将`0`值修改为`null`之外，其余的查询数据是一致的。

![mysql改进后的查询结果](/blog_img/devops/hidden-zero-value-in-grafana-bar-chart/mysql-query-result-2.png  "mysql改进后的查询结果")

在`Grafana`中查看显示效果类似如下，可发现0值已经消失，问题初步解决。

![柱状图不显示0值](/blog_img/devops/hidden-zero-value-in-grafana-bar-chart/bar-chart-without-zero-values.png  "柱状图不显示0值")

## 后记

虽然前述问题被解决，但当我们用鼠标查询对应的柱状条时，可发现`tooltip`中会显示所有的类别，即使该类别值为0或不存在也会显示，同样很影响使用体验。

在网上搜索一番后，发现已经有人在`Grafana`的`GitHub`主页上提出了对应的issue[^2], 遗憾的是该问题在2015年初被提出，到现在快10年了都没解决，`Grafana`官方的开发人员明确回复了没有该问题的修复计划，并且不认为其是一个issue，看来只能一直凑合用着！

![柱状图tooltip中显示所有类别](/blog_img/devops/hidden-zero-value-in-grafana-bar-chart/bar-chart-tooltip-with-zero-category.png  "柱状图tooltip中显示所有类别")

[^1]: https://community.grafana.com/t/hiding-0s-from-charts-when-data-is-not-all-0s/100820
[^2]: https://github.com/grafana/grafana/issues/1381