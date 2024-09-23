---
title: "在MyBatis接口中使用Optional进行非空判断"
date: 2020-02-13T14:40:19+08:00
lastmod: 2020-02-13T14:40:19+08:00
draft: false
keywords: ["mybatis","optional"]
description: "在MyBatis接口中使用Optional进行非空判断，避免冗余的If Else判断"
tags: ["mybatis"]
categories: ["Java编程","MyBatis系列"]
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

在进行`Java`开发时，[**NullPointerException**](https://docs.oracle.com/javase/8/docs/api/java/lang/NullPointerException.html)是一个很常见的异常，在`Java8`中提供了[**Optional**](https://www.oracle.com/technical-resources/articles/java/java8-optional.html)来避免此问题，而在[**MyBatis 3.5.0**](https://blog.mybatis.org/2019/01/mybatis-350-released.html)中也提供了对`Optional`的支持，本文简要叙述如何在`MyBatis`中使用`Optional`。

<!--more-->
完整代码参见[**spring-mybatis-example**](https://github.com/lucumt/spring-mybatis-example)

## 查询单条记录

由于在查询单条记录时，若数据库中不存在该记录，则会默认返回null，此时可用`Optional`进行避免。



首先确保`MyBatis`的版本不低于**3.5.0**，在mapper中可以仿照如下定义接口

```java
public interface BookMapper {

    Optional<BookModel> findById(Integer id);
}
```

之后在对应的service类中可以按照如下方式使用

```java
@Service
public class BookService {

    @Autowired
    private BookMapper bookMapper;

    public BookModel getBook(Integer id) {
        Optional<BookModel> result = bookMapper.findById(id);
        BookModel book = result.orElse(new BookModel());
        return book;
    }
}
```

执行下述单元测试代码

```java
@SpringBootTest
public class TestBookService {

    @Autowired
    private BookService bookService;

    @Test
    public void testGetBook() {
        BookModel book = bookService.getBook(3);
        Assertions.assertNotNull(book);
        System.out.println(book);
    }
}
```

其执行结果如下，可见`Optional`在Mapper接口中生效。

![Optional执行结果](/blog_img/mybatis/using-optional-in-mybatis-mapper/mybatis-optional-test-result.png "Optional执行结果") 

## 查询多条记录

当查询的结果为`List`时，即使没有数据`MyBatis`也会返回一个空的`List`，其size为0但不为null，此时不需要我们进行非空判断。

定义如下接口

```java
public interface BookMapper {
    
    List<BookModel> selectByType(Integer type);
}
```

mapper文件如下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lucumt.mapper.BookMapper">

    <select id="findById" parameterType="Integer" resultType="bookModel">
        SELECT id,name,price,type FROM book WHERE id=#{id}
    </select>

    <select id="selectByType" parameterType="integer" resultType="bookModel">
        SELECT id,name,price FROM book WHERE type=#{type}
    </select>

</mapper>
```

业务类代码

```java
@Service
public class BookService {

    @Autowired
    private BookMapper bookMapper;

    public List<BookModel> selectByType(Integer type){
        List<BookModel> bookList = bookMapper.selectByType(type);
        return bookList;
    }
}
```

执行下述单元测试代码时能测试通过，可见在查询集合列表是不需要判断是否为空。

```java
@SpringBootTest
public class TestBookService {

    @Autowired
    private BookService bookService;

    @Test
    public void testSelectByType() {
        List<BookModel> bookList = bookService.selectByType(3);
        Assertions.assertTrue(bookList != null);
        Assertions.assertTrue(bookList.size() == 0);
    }
}
```

![List执行结果](/blog_img/mybatis/using-optional-in-mybatis-mapper/mybatis-list-test-result.png "List执行结果") 

从上述试验可知，当获取的结果为`List`时，不使用`Optinal`关键字也能避免空指针问题。