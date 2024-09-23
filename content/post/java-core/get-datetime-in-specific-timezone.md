---
title: "利用JDK8时区常量进行时区相关操作"
date: 2021-06-08T16:01:47+08:00
lastmod: 2021-06-08T16:01:47+08:00
draft: false
keywords: ["java","timezone","硬编码"]
description: "简要介绍如何通过JDK8时区常量进行时区相关操作，以提高代码可维护性"
tags: ["java"]
categories: ["java编程"]
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

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

在进行`Java`编程时有时候会涉及到时区相关的操作，本文简要介绍一种通过使用`JDK`内置的时区常量来进行时区相关操作。

<!--more-->

## ZoneId使用

在涉及到时区的操作时，通常采用类似如下的代码以[**硬编码**](https://zh.wikipedia.org/zh-cn/%E5%AF%AB%E6%AD%BB)方式实现相关功能。

```java
LocalDateTime datetime1 = LocalDateTime.ofInstant(Instant.now(), ZoneId.of("Asia/Shanghai"));
System.out.println(datetime1);
LocalDateTime datetime2 = LocalDateTime.ofInstant(Instant.now(), ZoneId.of("America/Chicago"));
System.out.println(datetime2);
```

上述代码中涉及到的时区名称虽然可通过网络查询获得，但使用起来不是特别方便，有木有可能通过`JDK`内置的API实现呢？



通过查阅网络资料以及`JDK`的官方文档，最终找到了答案，那就是`ZoneId`! 在其[**官方文档**](https://docs.oracle.com/javase/8/docs/api/java/time/ZoneId.html)中有如下说明

![ZoneId描述信息](/blog_img/java-core/get-datetime-in-specific-timezone/zone-id-description.png "ZoneId描述信息")  

基于上述说明在`ZoneId`的源码中，可找到如下代码段，该代码段中内置了一些常用时区表示，通过它们可以用更简洁的方式实现。

```java
    public static final Map<String, String> SHORT_IDS;
    static {
        Map<String, String> map = new HashMap<>(64);
        map.put("ACT", "Australia/Darwin");
        map.put("AET", "Australia/Sydney");
        map.put("AGT", "America/Argentina/Buenos_Aires");
        map.put("ART", "Africa/Cairo");
        map.put("AST", "America/Anchorage");
        map.put("BET", "America/Sao_Paulo");
        map.put("BST", "Asia/Dhaka");
        map.put("CAT", "Africa/Harare");
        map.put("CNT", "America/St_Johns");
        map.put("CST", "America/Chicago");
        map.put("CTT", "Asia/Shanghai");
        map.put("EAT", "Africa/Addis_Ababa");
        map.put("ECT", "Europe/Paris");
        map.put("IET", "America/Indiana/Indianapolis");
        map.put("IST", "Asia/Kolkata");
        map.put("JST", "Asia/Tokyo");
        map.put("MIT", "Pacific/Apia");
        map.put("NET", "Asia/Yerevan");
        map.put("NST", "Pacific/Auckland");
        map.put("PLT", "Asia/Karachi");
        map.put("PNT", "America/Phoenix");
        map.put("PRT", "America/Puerto_Rico");
        map.put("PST", "America/Los_Angeles");
        map.put("SST", "Pacific/Guadalcanal");
        map.put("VST", "Asia/Ho_Chi_Minh");
        map.put("EST", "-05:00");
        map.put("MST", "-07:00");
        map.put("HST", "-10:00");
        SHORT_IDS = Collections.unmodifiableMap(map);
    }
```

基于上述代码段改进后的使用方式如下，可以看出相对于初始实现稍微简洁那么一丢丢~

```java
LocalDateTime datetime3 = LocalDateTime.ofInstant(Instant.now(), ZoneId.of(ZoneId.SHORT_IDS.get("CTT")));
System.out.println(datetime3);
LocalDateTime datetime4 = LocalDateTime.ofInstant(Instant.now(), ZoneId.of(ZoneId.SHORT_IDS.get("CST")));
System.out.println(datetime4);
```

## ZoneId分析

由于前述`ZoneId.SHORT_IDS`中的时区都是通过字符串表示的，在上网查询的情况下可通过如下代码获取各个时区与[**UTC偏移量**](https://zh.wikipedia.org/zh-cn/UTC%E5%81%8F%E7%A7%BB%E9%87%8F)的对应关系

```java
Map<String, String> zoneIdMap = ZoneId.SHORT_IDS;
zoneIdMap.forEach((k, v) -> {
    ZoneOffset offset = ZoneId.of(v).getRules().getStandardOffset(Instant.EPOCH);
    System.out.println(k + "\t\t" + offset + "\t\t" + v);
});
```

执行结果如下

```java
CTT		+08:00		Asia/Shanghai
ART		+02:00		Africa/Cairo
CNT		-03:30		America/St_Johns
PRT		-04:00		America/Puerto_Rico
PNT		-07:00		America/Phoenix
PLT		+05:00		Asia/Karachi
AST		-10:00		America/Anchorage
BST		+06:00		Asia/Dhaka
CST		-06:00		America/Chicago
EST		-05:00		-05:00
HST		-10:00		-10:00
JST		+09:00		Asia/Tokyo
IST		+05:30		Asia/Kolkata
AGT		-03:00		America/Argentina/Buenos_Aires
NST		+12:00		Pacific/Auckland
MST		-07:00		-07:00
AET		+10:00		Australia/Sydney
BET		-03:00		America/Sao_Paulo
PST		-08:00		America/Los_Angeles
ACT		+09:30		Australia/Darwin
SST		+11:00		Pacific/Guadalcanal
VST		+08:00		Asia/Ho_Chi_Minh
CAT		+02:00		Africa/Harare
ECT		+01:00		Europe/Paris
EAT		+03:00		Africa/Addis_Ababa
IET		-05:00		America/Indiana/Indianapolis
MIT		-11:00		Pacific/Apia
NET		+04:00		Asia/Yerevan
```

上述输出结果是无序的，不便于对比使用，可将代码改进如下以便其能按顺序输出

```java
Map<String, String> zoneIdMap = ZoneId.SHORT_IDS;
zoneIdMap.entrySet().stream()
    .sorted((c1, c2) -> {
        ZoneOffset o1 = ZoneId.of(c1.getValue()).getRules().getStandardOffset(Instant.EPOCH);
        ZoneOffset o2 = ZoneId.of(c2.getValue()).getRules().getStandardOffset(Instant.EPOCH);
        return o1.compareTo(o2);
    })
    .forEach(e -> {
        String value = e.getValue();
        ZoneOffset offset = ZoneId.of(value).getRules().getStandardOffset(Instant.EPOCH);
        System.out.println(e.getKey() + "\t\t" + offset + "\t\t" + value);
    });
```

改进后的输出结果如下，除了个别时区之外基本上都覆盖到了。

```java
NST		+12:00		Pacific/Auckland
SST		+11:00		Pacific/Guadalcanal
AET		+10:00		Australia/Sydney
ACT		+09:30		Australia/Darwin
JST		+09:00		Asia/Tokyo
CTT		+08:00		Asia/Shanghai
VST		+08:00		Asia/Ho_Chi_Minh
BST		+06:00		Asia/Dhaka
IST		+05:30		Asia/Kolkata
PLT		+05:00		Asia/Karachi
NET		+04:00		Asia/Yerevan
EAT		+03:00		Africa/Addis_Ababa
ART		+02:00		Africa/Cairo
CAT		+02:00		Africa/Harare
ECT		+01:00		Europe/Paris
AGT		-03:00		America/Argentina/Buenos_Aires
BET		-03:00		America/Sao_Paulo
CNT		-03:30		America/St_Johns
PRT		-04:00		America/Puerto_Rico
EST		-05:00		-05:00
IET		-05:00		America/Indiana/Indianapolis
CST		-06:00		America/Chicago
PNT		-07:00		America/Phoenix
MST		-07:00		-07:00
PST		-08:00		America/Los_Angeles
AST		-10:00		America/Anchorage
HST		-10:00		-10:00
MIT		-11:00		Pacific/Apia
```

`SHORT_IDS`中内置的时区总共有28条，但从上述输出结果来看，仍有个别时区遗漏在其中，在`ZoneId`的API文档中说明`SHORT_IDS`是来源于`TZDB 2005r`，不过我没有在网上找到这方面的详细资料。对于不在`SHORT_IDS`中的，可从[**List of tz database time zones**](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)中获取响应的时区然后通过硬编码方式实现。

