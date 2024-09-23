---
title: "利用Java8中的lambda来实现字符串的解析与分组"
date: 2022-11-15T16:20:57+08:00
lastmod: 2022-11-15T16:20:57+08:00
draft: false
keywords: ["Java","Java8","Map","Array","Lambda"]
description: "利用Java8中的lambda来实现字符串的解析与分组，供自己后续使用时参考"
tags: ["Java"]
categories: ["Java编程"]
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

在项目中某个请求的参数采用了较为复杂的规则拼接而成，在服务器端查询时需要将其解析成符合要求的`Map`与`List`组合的格式。一开始自己采用的是传统的遍历方式实现，后续发现用`Java8`中引入的`lambda`表达式后代码变的更简洁、更优雅，故简单记录下。

<!--more-->

## 问题描述

项目中有类似如下的字符串输入参数

```
010$$fengtai,010$$chaoyang,010$$haidain,027$$wuchang,027$$hongshan,027$$caidan,021$$changnin,021$$xuhui,020$$tianhe
```

调用方期望将输入参数转化为如下格式(即首先根据`,`进行分割，然后在根据`$$`进行进一步的拆分，最终的结果是按照`$$`前面的数据为key，`$$`后面的数据分别放入同一个数组中，最终结果以`Map<String, List<String>>`的格式返回)。

```
{027=[wuchang, hongshan, caidan], 020=[tianhe], 010=[fengtai, chaoyang, haidain], 021=[changnin, xuhui]}
```

## 传统方式

最开始由于需要尽快部署调试，自己才用的是传统的遍历方式，此种方式编码起来很简单，具备简单的`Java`基础即可实现。

```java
public Map<String, List<String>> parseParametersByIterate(String sensors) {
    List<String[]> dataList = Arrays.stream(sensors.split(",")).map(s -> s.split("\\$\\$")).collect(Collectors.toList());
    Map<String, List<String>> resultMap = new HashMap<>();
    for (String[] d : dataList) {
        List<String> list = resultMap.get(d[0]);
        if (list == null) {
            list = new ArrayList<>();
            list.add(d[1]);
            resultMap.put(d[0], list);
        } else {
            list.add(d[1]);
        }
    }
    return resultMap;
}
```

## 基于Lambda实现

在[Stack Overflow](https://stackoverflow.com/questions/74443495/how-to-group-and-map-string-via-lambda)上咨询后，发现用`lambda`实现更简洁，只用一行代码就能搞定!

```java
public Map<String, List<String>> parseByLambda(String data) {
    Map<String, List<String>> resultMap = Arrays.stream(data.split(","))
        .map(s -> s.split("\\$\\$"))
        .collect(Collectors.groupingBy(s -> s[0],
                                       Collectors.mapping(s -> s[1], Collectors.toList())));
    return resultMap;
}
```

上述代码的核心是采用Java8中引入的[Collectors](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html)类，采用其中的`groupingBy`和`mapping`方法实现，而自己用的最多的只是`toList`和`toMap`，基本功有待加强！