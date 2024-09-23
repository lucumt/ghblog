---
title: "Kafka每次都消费最新的数据"
date: 2023-11-16T15:42:25+08:00
lastmod: 2023-11-16T15:42:25+08:00
draft: false
keywords: ["kafka","最新数据"]
description: "简要介绍如何以多种方式实现Kafka每次都消费最新的数据"
tags: ["kafka"]
categories: ["消息队列"]
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

近期项目中有一个数据展示功能，其数据来源于[**Kafka**](https://kafka.apache.org/)，项目要求`Kafka`每次都消费最新的数据，简要记录下其实现方案。

<!--more-->

## 固定组实现-不生效

将`auto.offset.reset`设置为`latest`同时采用固定group，**此种方式只在第一次读取时有效，后续再次读取时仍然从上次读取的地方开始继续读**，不满足使用要求。

```java
Properties props = new Properties();
props.put("bootstrap.servers", "10.10.2.98:9004");
// 每次将组名动态生成
props.put("group.id", "test-group-1");
props.put("auto.commit.interval.ms", "1000");
props.put("session.timeout.ms", "30000");
props.put("auto.offset.reset", "latest");
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

String topic = "raw_message";
consumer.subscribe(Arrays.asList(topic));
```

## 动态组实现-生效

将`auto.offset.reset`设置为`latest`，同时每次消费时将group的名称动态生成，这样即可确保每次读取的都是最新的消息。

其缺点是group的数量会不断增加，偏移量___consumer_offsets 多次保存，且没找到有效的删除group的方法，也没办法做到定期清理，会对性能产生影响，通常只作为测试与实现使用。

```java
Properties props = new Properties();
props.put("bootstrap.servers", "10.10.2.98:9004");
// 每次将组名动态生成
props.put("group.id", "test-group-" + RandomStringUtils.randomAlphabetic(6));
props.put("auto.commit.interval.ms", "1000");
props.put("session.timeout.ms", "30000");
props.put("auto.offset.reset", "latest");
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

String topic = "raw_message";
consumer.subscribe(Arrays.asList(topic));
```

## 固定组实现-生效

将`auto.offset.reset`设置为`latest`的同时，group名称固定不变 ，给对应Consumer调用[**seekToEnd()**](https://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#seekToEnd(java.util.Collection))方法，此种方式不需要动态切换组，推荐使用此方式。

```java
Properties props = new Properties();
props.put("bootstrap.servers", "10.10.2.98:9004");
props.put("group.id", "test-group-1");
props.put("auto.commit.interval.ms", "1000");
props.put("session.timeout.ms", "30000");
props.put("auto.offset.reset", "latest");
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

String topic = "raw_message";
TopicPartition topicPartition = new TopicPartition(topic, 0);
List<TopicPartition> topics = Arrays.asList(topicPartition);
consumer.assign(topics);
consumer.seekToEnd(topics);
```

