---
title: "CompletableFuture异步方法使用时遇到的问题"
date: 2022-08-29T09:35:29+08:00
lastmod: 2022-08-29T09:35:29+08:00
draft: true
keywords: []
description: ""
tags: ["Java","Java Concurrency"]
categories: []
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

自己尚未找出原因

<!--more-->

```java
public class CompletableFutureTest {

    public static void main(String[] args) {
        test1();
    }

    private static void test1() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("HH:mm:ss:SSS");
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        List<Integer> dataList = Arrays.asList(1, 2, 3);
        for (Integer data : dataList) {
            CompletableFuture.supplyAsync(() -> {
                try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                String result = LocalDateTime.now().format(formatter) + "\t结果\t" + data;
                return result;
            }, executorService).whenCompleteAsync((result, exception) -> {
                System.out.println(result);
            });
        }
        executorService.shutdown();

        try {
            String startTime = "Started at:\t" + LocalDateTime.now().format(formatter);
            System.out.println(startTime);
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

