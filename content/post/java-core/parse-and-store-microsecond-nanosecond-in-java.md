---
title: "在Java中解析和存储包含微秒与纳秒的时间"
date: 2023-02-17T13:48:05+08:00
lastmod: 2023-02-17T13:48:05+08:00
draft: false
keywords: ["Java","==","equals"]
description: "简要描述如何在Java中解析和存储包含微秒与纳秒的时间"
tags: ["Java"]
categories: ["Java编程"]
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
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true

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

在软件开发领域，大部分时间精确到秒或毫秒即可满足日常需求，但在某些对时间要求严格的场景中需要使用微秒、纳秒等更精确的时间值，本文简要记录如何在Java中通过[LocalDateTime](https://docs.oracle.com/javase/8/docs/api/java/time/LocalDateTime.html)实现对于微秒、纳秒的精确解析以及转化为`long`型时间戳。

<!--more-->

## java.util.Date中相关实现

### 格式化测试

首先采用采用如下代码验证`java.util.Date`和`java.text.SimpleDateFormat`对于时间戳的支持情况。

```java
public class TestDateConvert1 {

    public static void main(String[] args) {
        String text1 = "2023/01/04 17:54:38";
        String text2 = "2023/01/04 17:54:38.610";
        String text3 = "2023/01/04 17:54:38.610502";
        String text4 = "2023/01/04 17:54:38.610502567";
        try {
            testDate1(text1);
            testDate2(text2);
            testDate3(text3);
            testDate4(text4);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }

    public static void testDate1(String text) throws ParseException {
        DateFormat df = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss"); //精确到秒
        Date newDate = df.parse(text);
        System.out.println("----------------精确到秒------------------------------");
        System.out.println("原始时间:\t" + text);
        System.out.println("解析后的时间:\t" + df.format(newDate));
        System.out.println();
    }

    public static void testDate2(String text) throws ParseException {
        DateFormat df = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss.SSS"); // 精确到毫秒
        Date newDate = df.parse(text);
        System.out.println("----------------精确到毫秒----------------------------");
        System.out.println("原始时间:\t" + text);
        System.out.println("解析后的时间:\t" + df.format(newDate));
        System.out.println();
    }

    public static void testDate3(String text) throws ParseException {
        DateFormat df = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss.SSSSSS"); //精确到微秒
        Date newDate = df.parse(text);
        System.out.println("----------------精确到微秒----------------------------");
        System.out.println("原始时间:\t" + text);
        System.out.println("解析后的时间:\t" + df.format(newDate));
        System.out.println();
    }

    public static void testDate4(String text) throws ParseException {
        DateFormat df = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss.SSSSSSSSS"); //精确到微秒
        Date newDate = df.parse(text);
        System.out.println("----------------精确到纳秒----------------------------");
        System.out.println("原始时间:\t" + text);
        System.out.println("解析后的时间:\t" + df.format(newDate));
    }

}
```

运行结果如下：

![不同时间格式下的Date输出](/blog_img/java-core/parse-and-store-microsecond-in-java/date-format-test-1.png "不同时间格式下的Date输出")  

从上图中可以看出时间精确到微秒之后，采用`java.util.Date`和`java.text.SimpleDateFormat`进行输出时前后的结果已经不一致。同时也可以大致猜测，当使用`java.util.Date`时不能用其进行微秒或纳秒等高精度的时间存储展示，但此问题是由`java.util.Date`还是`java.text.SimpleDateFormat`造成的暂不确定。

### 对比数据测试

为了测试结果的准确性，在[www.timestamp-converter.com](https://www.timestamp-converter.com/)中获取一个可用于验证的时间戳信息如下所示

![测试用的时间戳](/blog_img/java-core/parse-and-store-microsecond-in-java/timestamp-test-data.png "测试用的时间戳")  

其中基于毫秒的时间戳为`1676628010725`，格式化后的显示为`2023-02-17T10:00:10.725Z`，采用下述代码进行验证：

```java
public static void testDateCreate() {
    DateFormat df1 = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
    DateFormat df2 = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss.SSS");
    DateFormat df3 = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss.SSSSSS");
    DateFormat df4 = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss.SSSSSSSSS");
    long timestamp1 = 1676628010725L;//前述获取到的时间戳
    Date date = new Date(timestamp1);
    long timestamp2 = date.getTime();
    System.out.println("timestamp1:\t" + timestamp1);
    System.out.println("timestamp2:\t" + timestamp2);
    System.out.println("精确到秒:\t" + df1.format(date));
    System.out.println("精确到毫秒:\t" + df2.format(date));
    System.out.println("精确到微秒:\t" + df3.format(date));
    System.out.println("精确到纳秒:\t" + df4.format(date));
}
```

运行结果如下

![Date对比测试结果](/blog_img/java-core/parse-and-store-microsecond-in-java/date-format-test-2.png "Date对比测试结果")  

从结果中可知微秒与纳秒的结果与网站中展示的不一致，但是仍然无法确定到底是`java.util.Date`还是`java.text.SimpleDateFormat`的原因。



在`java.util.Date`的[官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/Date.html#Date--)中可找到如下说明，从图中可知当采用时间戳来构造`java.util.Date`时，其接收的参数为毫秒相对数据值，不支持微秒和纳秒。 

![java.util.Date构造方法说明](/blog_img/java-core/parse-and-store-microsecond-in-java/java-util-date-constuctor.png "java.util.Date构造方法说明") 

在`java.text.SimpleDateFormate`的[官方文档](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html)中有如下说明，同样可知道`java.text.SimpleDateFormat`不支持微秒和纳秒级别的时间精度。

![java.text.SimpleDateFormat日期时间格式说明](/blog_img/java-core/parse-and-store-microsecond-in-java/javat-text-simpledateformat-pattern.png "java.text.SimpleDateFormat日期时间格式说明") 



**初步结论**:

> **`java.util.Date`和`java.text.SimpleDateFormat`均不支持微秒和纳秒级别的时间精度。**

## LocalDateTime中实现

由于`JDK8`中引入了`LocalDateTime`，故将最开始的测试代码都修改为使用`java.time.LocalDateTime`和`java.time.format.DateTimeFormatter`进行测试

```java
public class TestDateConvert3 {

    public static void main(String[] args) {
        String text1 = "2023/01/04 17:54:38";
        String text2 = "2023/01/04 17:54:38.610";
        String text3 = "2023/01/04 17:54:38.610502";
        String text4 = "2023/01/04 17:54:38.610502567";
        try {
            testDate1(text1);
            testDate2(text2);
            testDate3(text3);
            testDate4(text4);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }

    public static void testDate1(String text) throws ParseException {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss"); //精确到秒
        LocalDateTime newDate = LocalDateTime.parse(text,formatter);
        System.out.println("----------------精确到秒------------------------------");
        System.out.println("原始时间:\t" + text);
        System.out.println("解析后的时间:\t" + formatter.format(newDate));
        System.out.println();
    }

    public static void testDate2(String text) throws ParseException {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss.SSS"); //精确到毫秒
        LocalDateTime newDate = LocalDateTime.parse(text,formatter);
        System.out.println("----------------精确到毫秒----------------------------");
        System.out.println("原始时间:\t" + text);
        System.out.println("解析后的时间:\t" + formatter.format(newDate));
        System.out.println();
    }

    public static void testDate3(String text) throws ParseException {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss.SSSSSS"); //精确到微秒
        LocalDateTime newDate = LocalDateTime.parse(text,formatter);
        System.out.println("----------------精确到微秒----------------------------");
        System.out.println("原始时间:\t" + text);
        System.out.println("解析后的时间:\t" + formatter.format(newDate));
        System.out.println();
    }

    public static void testDate4(String text) throws ParseException {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss.SSSSSSSSS"); //精确到纳秒
        LocalDateTime newDate = LocalDateTime.parse(text,formatter);
        System.out.println("----------------精确到纳秒----------------------------");
        System.out.println("原始时间:\t" + text);
        System.out.println("解析后的时间:\t" + formatter.format(newDate));
        System.out.println();
    }

}
```

测试结果如下：

![java.time.LocalDateTime测试结果](/blog_img/java-core/parse-and-store-microsecond-in-java/localdatetime-format-test-1.png "java.time.LocalDateTime测试结果")  

从中可知当联合采用`java.time.LocalDateTime`和`java.time.format.DateTimeFormatter`时，能够支持到纳秒级别，可满足要求。

在`java.time.LocalDateTime`的[官方文档](https://docs.oracle.com/javase/8/docs/api/java/time/LocalDateTime.html)中有如下说明，从图中可知`java.time.LocalDateTime`支持到纳秒级别的时间精度

![java.time.LocalDateTime精确度说明](/blog_img/java-core/parse-and-store-microsecond-in-java/localdatetime-precision-instruction.png "java.time.LocalDateTime精确度说明")  

在`java.time.format.DateTimeFormatter`的[官方文档](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)中有如下说明,从图中可知`java.time.format.DateTimeFormatter`也支持纳秒级别的格式化。

![java.time.format.DateTimeFormatter格式化类型说明](/blog_img/java-core/parse-and-store-microsecond-in-java/datetimeformatter-pattern-instruction.png "java.time.format.DateTimeFormatter格式化类型说明") 

**最终结论**:

> **`java.time.LocalDateTime`支持纳秒级别的时间存储，支持`java.time.format.DateTimeFormatter`支持纳秒级别的时间显示。**

## LocalDateTime与时间戳互转

在`java.util.Date`中可以很容易的通过`Date().getTime()`和`new Date(long timestamp)`来分别获取时间戳和基于时间戳构造时间，而在`java.time.LocalDateTime`中要实现类似功能则稍微复杂点。



当要获取long类型的时间戳时，需要先获取秒，再获取微秒或纳秒，然后根据他们之前的换算关系进行累加返回，相关公式如下：

> * $微秒时间戳=秒 \times 10^6 +微秒$
> * $纳秒时间戳=秒 \times 10^9 +纳秒$

有了时间戳之后根据上述公式进行反向操作，分别获取秒与微秒(纳秒)，然后根据这2个数值通过`Instant`构建即可。

### 与微秒互转

```java
public class TestDateConvert4 {

    public static void main(String[] args) {
        long time = convertTimeToMills("2023/01/04 17:54:38.610502");
        convertMillsToTime(time);
    }

    public static long convertTimeToMills(String originalText) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss.SSSSSS");
        System.out.println(originalText);
        LocalDateTime dateTime = LocalDateTime.parse(originalText, formatter);
        ZoneId zoneId = ZoneId.systemDefault();
        Instant instant = dateTime.atZone(zoneId).toInstant();

        long seconds = instant.getEpochSecond();
        int micros = instant.get(ChronoField.MICRO_OF_SECOND);
        long total = seconds * 1_000_000 + micros;
        return total;
    }

    public static LocalDateTime convertMillsToTime(long time) {
        long sec = time / 1_000_000;
        long mic = time % 1_000_000;
        Instant instant1 = Instant.ofEpochSecond(sec).plus(mic, ChronoUnit.MICROS);
        ZoneId zoneId = ZoneId.systemDefault();
        LocalDateTime localDateTime = LocalDateTime.ofInstant(instant1, zoneId);

        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss.SSSSSS");
        System.out.println(formatter.format(localDateTime));

        return localDateTime;
    }
    
}
```

测试结果如下，符合预期

![微秒级别的时间戳转换](/blog_img/java-core/parse-and-store-microsecond-in-java/convert-to-millis-timestamp-test.png "微秒级别的时间戳转换")  

### 与纳秒互转

```java
public class TestDateConvert5 {

    public static void main(String[] args) {
        long time = convertTimeToMills("2023/01/04 17:54:38.610502987");
        convertMillsToTime(time);
    }

    public static long convertTimeToMills(String originalText) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss.SSSSSSSSS");
        System.out.println(originalText);
        LocalDateTime dateTime = LocalDateTime.parse(originalText, formatter);
        ZoneId zoneId = ZoneId.systemDefault();
        Instant instant = dateTime.atZone(zoneId).toInstant();

        long seconds = instant.getEpochSecond();
        int micros = instant.get(ChronoField.NANO_OF_SECOND);
        long total = seconds * 1_000_000_000 + micros;
        return total;
    }

    public static LocalDateTime convertMillsToTime(long time) {
        long sec = time / 1_000_000_000;
        long mic = time % 1_000_000_000;
        Instant instant1 = Instant.ofEpochSecond(sec).plus(mic, ChronoUnit.NANOS);
        ZoneId zoneId = ZoneId.systemDefault();
        LocalDateTime localDateTime = LocalDateTime.ofInstant(instant1, zoneId);

        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss.SSSSSSSSS");
        System.out.println(formatter.format(localDateTime));

        return localDateTime;
    }

}
```

测试结果如下，符合预期

![纳秒级别的时间戳转换](/blog_img/java-core/parse-and-store-microsecond-in-java/convert-to-nanos-timestamp-test.png "纳秒级别的时间戳转换")  

参考文章：

* https://www.cnblogs.com/yangxunwu1992/p/5769533.html
* https://stackoverflow.com/questions/30135025/java-date-parsing-with-microsecond-or-nanosecond-accuracy

