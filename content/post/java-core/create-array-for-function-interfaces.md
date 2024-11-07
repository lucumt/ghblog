---
title: "使用MyBatis-Plus时给SFunction接口创建属性数组以简化代码"
date: 2024-08-21T19:51:08+08:00
lastmod: 2024-08-21T19:51:08+08:00
draft: true
keywords: ["java","泛型","数组","MyBatis","MyBatis Plus"]
description: "通过一个简单的实例介绍如何在使用MyBatis-Plus时给SFunction接口创建属性数组以简化代码"
tags: ["java","mybatis"]
categories: ["java编程"]
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

简要

<!--more-->

## 问题

实体类源码如下

```java
import lombok.Data;

@Data
public class User {
    private Long id;
    private String name;
    private int age;
    private String email;
    private String department;
    private String phone;
}
```

接口类源码如下

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.lucumt.entity.User;

public interface UserMapper extends BaseMapper<User> {
}
```

基于`MyBatis-Plus`的查询代码如下，可以看出第

```java
public class UserMapperTest {

    @Resource
    private UserMapper userMapper;

    @Test
    public void testQueryUsers() {
        List<User> userList = userMapper.selectList(Wrappers.<User>lambdaQuery().select(User::getId, User::getName, User::getAge, User::getEmail, User::getPhone, User::getDepartment));
        Assertions.assertNotNull(userList);
    }
}
```

## 分析

```bash
echo "test"
```

## 实现