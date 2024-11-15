---
title: "使用MyBatis-Plus时给SFunction接口创建属性数组以简化代码"
date: 2024-08-21T19:51:08+08:00
lastmod: 2024-08-21T19:51:08+08:00
draft: false
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

通过一个简单的实例介绍如何在使用[MyBatis-Plus](https://baomidou.com)时给[SFunction](https://javadoc.io/doc/com.baomidou/mybatis-plus-core/3.4.3/com/baomidou/mybatisplus/core/toolkit/support/SFunction.html)接口创建属性数组以简化代码，同时巩固下自己的`Java`基础知识。 

<!--more-->

## 背景

实体类`User`源码如下

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

接口类`UserMapper`源码如下，其集成了`MyBatis-Plus`的基础接口，以便后续可使用`MyBatis-Plus`进行快速开发。

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.lucumt.entity.User;

public interface UserMapper extends BaseMapper<User> {
}
```

对应的测试代码如下，需要查询`User`类中的相关属性，可以看出第8行的代码长度很长，出现了滚动条导致阅读起来不太方便

```java { data-line="8" }
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

虽然可以通过换行来消除滚动条，但本质上还是一行代码，当该类的属性很多时，若需要实现类似`SELECT *`的效果则会导致多个换行，还是不太便于阅读理解。

```java { data-line="8-9" }
public class UserMapperTest {

    @Resource
    private UserMapper userMapper;

    @Test
    public void testQueryUsers() {
        List<User> userList = userMapper.selectList(Wrappers.<User>lambdaQuery().select(User::getId, User::getName,
                User::getAge, User::getEmail, User::getPhone, User::getDepartment));
        Assertions.assertNotNull(userList);
    }
}
```

自己期望将这些属性都放到一个数组中，在实现简化的同时也能便于阅读理解。

## 尝试

最开始自己很直接的想把上述属性提取到一个数组中，将代码修改如下

```java { data-line="8-10" }
public class UserMapperTest {

    @Resource
    private UserMapper userMapper;

    @Test
    public void testQueryUsers() {
        SFunction<User, ?>[] properties = new SFunction[]{User::getId, User::getName,
                User::getAge, User::getEmail, User::getPhone, User::getDepartment};
        List<User> userList = userMapper.selectList(Wrappers.<User>lambdaQuery().select(properties));
        Assertions.assertNotNull(userList);
    }
}
```

修改完毕后编译器立即显示编译错误，直接放置到数据中的方式不通。

 ![代码编译错误](/blog_img/java-core/create-array-for-function-interfaces/property-arrray-compile-error.png "代码编译错误")

将其修改为如下后，编译错误消失，此时虽然实现了将它们都放到数组中，但是每个属性都需要进行变量声明，代码篇幅较多，不是自己期望的最优解。

```java { data-line="15" }
public class UserMapperTest {

    @Resource
    private UserMapper userMapper;

    @Test
    public void testQueryUsers() {
        SFunction<User, ?> getId = User::getId;
        SFunction<User, ?> getName = User::getName;
        SFunction<User, ?> getAge = User::getAge;
        SFunction<User, ?> getEmail = User::getEmail;
        SFunction<User, ?> getPhone = User::getPhone;
        SFunction<User, ?> getDepartment = User::getDepartment;
        SFunction<User, ?>[] properties = new SFunction[]{getId, getName, getAge, getEmail, getPhone, getDepartment};
        List<User> userList = userMapper.selectList(Wrappers.<User>lambdaQuery().select(properties));
        Assertions.assertNotNull(userList);
    }
}
```

## 分析

为啥第一种方式方式会报错，而第二种方式却能正常工作呢？

自己虽然知道这和`Java`和[泛型](https://docs.oracle.com/javase/tutorial/java/generics/index.html)相关的[类型擦除](https://zh.wikipedia.org/wiki/%E7%B1%BB%E5%9E%8B%E6%93%A6%E9%99%A4)，但限于自己孱弱的`Java`基础知识，并不能给出一个很好的解释。

我决定寻求更专业的协助，在`Stackoverflow` 上提出了[一个问题](https://stackoverflow.com/questions/78763705/can-not-create-function-array-in-java-8)，有2个大佬都从[Java语言规范](https://docs.oracle.com/javase/specs/)的角度给出了让人信服的回答。

首先前面图中出现的错误不是根本原因，基于我最开始的写法会导致多个错误，而IDE只会展示最顶层的错误，由此让人很迷惑，实际的原因是`类型擦除`。

在[Array Initializers](https://docs.oracle.com/javase/specs/jls/se22/html/jls-10.html#jls-10.6)中有如下说明

> Each variable initializer must be assignment-compatible (§5.2) with the array's component type, or a compile-time error occurs.

翻译成人话就是数组中的每一个元素的类型都必须与数组类型兼容，否则会出现编译错误。

 前述的数组创建方式

```java
SFunction<User, ?>[] properties = new SFunction[]{User::getId, User::getName, 
                   User::getAge, User::getEmail, User::getPhone, User::getDepartment};
```

实际上等价于下述代码

```java { data-line="1-6" }
SFunction getId = User::getId;
SFunction getName = User::getName;
SFunction getAge = User::getAge;
SFunction getEmail = User::getEmail;
SFunction getPhone = User::getPhone;
SFunction getDepartment = User::getDepartment;
SFunction<User, ?>[] properties = new SFunction[]{getId, getName, getName, getEmail, getPhone, getDepartment};
```

而上述6行代码是无法编译通过的，其错误信息和前面截图一样，这是代码层面的问题溯源。



为什么那6行代码无法编译呢？需要在`Java语言规范`的[Function Types](https://docs.oracle.com/javase/specs/jls/se22/html/jls-9.html#jls-9.9)中去寻找答案

> The function type of the raw type of a generic functional interface I<...> is the erasure of the function type of the generic functional interface I<...>.

在[这个回答](https://stackoverflow.com/a/78767296/3176419)中回答者结合上述说明给出了详细的解释：

由于存在继承关系，通用的`SFunction<T, R>`函数类型与[Function<T, R>](https://docs.oracle.com/javase/8/docs/api/?java/util/function/Function.html)相同，其利用`T`作为输入，`R`作为输出，可以简单记做`T->R`。由于`T`和`R`都没有进行类型约束(即通过`extends`或`super`关键字进行约束)，故它们都将被类型擦除为默认的`Object`类型，故`SFunction<T, R>`的函数类型实际上为`Object->Object`。

回到我们的问题中来`User::getId`是否兼容`Object->Object`呢？显然不是，其输入为 `User`，输出为`Integer`，实际的函数类型为`User->Integer`，故而不兼容。

基于上述分析以及对应回答者的建议，有如下几种方式：

1. 修改源码以兼容函数类型

   ```java
   Function<? super Object, ? extends Object> properties = User::getId;
   ```

2. 采用`List`代替数组

   ```java
   List<SFunction<User, ?>> propertyList = Lists.newArrayList(User::getId, User::getName);
   ```

3. 显示的指定为实际的类型

   ```java
   // 通过变量实现
   SFunction<User, ?>[] properties = new SFunction[] {val -> ((User) val).getId(),  val -> ((User) val).getName() };
   
   // 方法引用实现
   SFunction<User, ?>[] propertiesss = new SFunction[]{(SFunction<User, ?>) User::getId, (SFunction<User, ?>) User::getName};   
   ```

4. 编写一个特定的方法用于进行类型转化

   ```java
   <T, R> SFunction<T, R> makeFunction(SFunction<T, R> x) { return x; }
   ```

## 改进

前面的4种方式，第1种由于需要修改源码不现实，故不考虑，第2种是修改数据类型与`MyBatis-Plus`的方法签名不匹配，也不考虑。

采用第3种方式可将代码修改如下，此种方式与之前的差别不大，还是无法避免强制转换

```java
public class UserMapperTest {

    @Resource
    private UserMapper userMapper;

    @Test
    public void testQueryUsers() {
        SFunction<User, ?>[] properties = new SFunction[]{
                (SFunction<User, ?>) User::getId,
                (SFunction<User, ?>) User::getName,
                (SFunction<User, ?>) User::getAge,
                (SFunction<User, ?>) User::getEmail,
                (SFunction<User, ?>) User::getPhone,
                (SFunction<User, ?>) User::getDepartment
        };
        List<User> userList = userMapper.selectList(Wrappers.<User>lambdaQuery().select(properties));
        Assertions.assertNotNull(userList);
    }
}
```

第4种方式相较于第3种只是提供了一个额外的方法进行统一的类型转化，但本质上和之前的一样

```java
public class UserMapperTest {

    @Resource
    private UserMapper userMapper;

    @Test
    public void testQueryUsers() {
        SFunction<User, ?>[] properties = new SFunction[]{
                getProperty(User::getId),
                getProperty(User::getName),
                getProperty(User::getAge),
                getProperty(User::getEmail),
                getProperty(User::getPhone),
                getProperty(User::getDepartment)
        };
        List<User> userList = userMapper.selectList(Wrappers.<User>lambdaQuery().select(properties));
        Assertions.assertNotNull(userList);
    }

    private <T, R> SFunction<T, R> getProperty(SFunction<T, R> x) {
        return x;
    }
}
```

## 后记

前述问题的解决主要是依赖于`Java语言规范`，而自己却一直不怎么关注，我想起了另外一件事情。

虽然自己的在`Stackoverflow`上的[积分也很多](https://stackoverflow.com/users/3176419/flyingfox)，但实话实说其中的大部分都是我在`ChatGPT`没有出来之前通过回答大量的`JavaScript`水出来的

 ![个人Stackoverflow积分](/blog_img/java-core/create-array-for-function-interfaces/my-stackoverflow-analysis.png "个人Stackoverflow积分")

如上图所示，截止到当前个人在`Stackoverflow` 上的总积分为13496，而自己回答了729个问题，排除掉自己提问带来的积分，平均每个回答带来的积分只有`18.4`，完全是数量压倒质量(quantity over quality)。

具体到相关问题和回答时，自己的表现也一般，如下图所示，得分最高的回答也才10个赞，只贡献了50分。

 ![个人Stackoverflow高赞回答](/blog_img/java-core/create-array-for-function-interfaces/my-stackoverflow-top-reputation.png "个人Stackoverflow高赞回答")

而自己关注的另一位[大佬](https://stackoverflow.com/users/6690200/xingbin)的回答概况如下，其平均得分是我的两倍

![他人的Stackoverflow积分](/blog_img/java-core/create-array-for-function-interfaces/other-stackoverflow-analysis.png "他人的Stackoverflow积分")

在具体问题和回答上的积分更是碾压我!

 ![他人Stackoverflow高赞回答](/blog_img/java-core/create-array-for-function-interfaces/other-stackoverflow-top-reputation.png "他人Stackoverflow高赞回答")

深入查看后可发现该大佬很多问题都是基于`Java语言规范`回答的，此种回答言简意赅，直击问题并且很容易获得很多赞，从而快速提高在`Stackoverflow`中的积分。

 ![基于java规范回答问题](/blog_img/java-core/create-array-for-function-interfaces/answer-question-with-jls.png "基于java规范回答问题")

要想获得类似上面大佬一样的成就，必须要熟练掌握`Java语言规范`，而我们日常学习与开发中，更多关注的是`Java`的基本语法与使用场景，对`Java语言规范`则很少有人深入学习与研究。

结合当前`Java`就业市场的内卷以及大的环境，如果不能深入钻研并精通某一方面，很容易遭到淘汰，我们不仅要提高知识面的广度，更要熟练掌握某一个领域才能在残酷的竞争中立足。

**Quality over Quantity!**