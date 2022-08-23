---
title: "MyBatis中插入JSON格式数据"
date: 2021-04-15T09:37:41+08:00
lastmod: 2021-04-15T09:37:41+08:00
draft: false
keywords: ["Java","MyBatis","JSON"]
description: ""
tags: ["Java","MyBatis"]
categories: ["Java编程","MyBatis系列"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
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

简单记录下如何在[**MyBatis**](https://mybatis.org/mybatis-3/)中插入[**JSON**](https://www.json.org/json-en.html)，以备参考。

<!--more-->

#  基本信息

完整代码参见[**spring-mybatis-example**](https://github.com/lucumt/spring-mybatis-example)，核心代码为[**JsonNodeTypeHandler**](#handler类)配置类。

## 项目结构

![项目结构](/blog_img/mybatis/insert-json-data-in-mybatis/spring-mybatis-project-structure.png "项目结构") 

## 表结构

```sql
CREATE TABLE `user_info` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) COLLATE utf8_bin NOT NULL,
  `info` json NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
```

## pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lucumt</groupId>
    <artifactId>spring-mybatis-test</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>SpringTest</name>
    <url>https://lucumt.info</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <spring.boot.version>2.7.3</spring.boot.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>${spring.boot.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-launcher</artifactId>
            <version>1.9.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.13.3</version>
        </dependency>

    </dependencies>

</project>
```

## 模型类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserModel {

    private Integer id;

    private String name;

    private JsonNode info;
}
```

## Handler类

```java
@MappedTypes(JsonNode.class)
public class JsonNodeTypeHandler extends BaseTypeHandler<JsonNode> {

    /**
     * 设置非空参数
     *
     * @param ps
     * @param i
     * @param parameter
     * @param jdbcType
     * @throws SQLException
     */
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, JsonNode parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter.toString());
    }

    /**
     * 根据列名，获取可以为空的结果
     *
     * @param rs
     * @param columnName
     * @return
     * @throws SQLException
     */
    @Override
    public JsonNode getNullableResult(ResultSet rs, String columnName) throws SQLException {
        String sqlJson = rs.getString(columnName);
        return getResultFromJSON(sqlJson);
    }

    /**
     * 根据列索引，获取可以为内控的接口
     *
     * @param rs
     * @param columnIndex
     * @return
     * @throws SQLException
     */
    @Override
    public JsonNode getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        String sqlJson = rs.getString(columnIndex);
        return getResultFromJSON(sqlJson);
    }

    /**
     * @param cs
     * @param columnIndex
     * @return
     * @throws SQLException
     */
    @Override
    public JsonNode getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        String sqlJson = cs.getNString(columnIndex);
        return getResultFromJSON(sqlJson);
    }

    private JsonNode getResultFromJSON(String sqlJson) {
        if (sqlJson == null) {
            return null;
        }
        try {
            return new ObjectMapper().readTree(sqlJson);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
    }

}
```

## 配置文件

```yaml
server:
  port: 8081

logging:
  level:
    root: info

spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://10.10.2.98:3316/db_test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    username: root
    password: 654321

mybatis:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
  type-aliases-package: com.lucumt.model,com.lucumt.vo
  type-handlers-package: com.lucumt.handler.mybatis
```

## Mapper类

```java
public interface UserMapper {

    void addUser(UserModel userModel);

    UserModel findById(Integer id);

    void deleteUser(Integer id);
}
```

## Mapper文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lucumt.mapper.UserMapper">

    <select id="findById" parameterType="Integer" resultType="userModel">
        SELECT id,name,info FROM user_info WHERE id=#{id}
    </select>

    <insert id="addUser" parameterType="userModel" useGeneratedKeys="true" keyProperty="id" keyColumn="id">
        INSERT INTO user_info(name,info) VALUES
        (#{name},#{info,typeHandler=com.lucumt.handler.mybatis.JsonNodeTypeHandler})
    </insert>

    <delete id="deleteUser" parameterType="Integer">
        DELETE FROM user_info WHERE id= #{id}
    </delete>

</mapper>
```

## Service类

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public void addUser(UserModel userModel) {
        userMapper.addUser(userModel);
    }

    public UserModel getUser(Integer id) {
        return userMapper.findById(id);
    }

    public void deleteUser(Integer id) {
        userMapper.deleteUser(id);
    }
}
```

# 测试

## 测试类

```java
@SpringBootTest
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class TestUserService {

    @Autowired
    private UserService userService;

    private static Integer userId;

    @Order(1)
    @Test
    public void testAddUser() throws JsonProcessingException {
        String time = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd_HH:mm:ss:SSS"));
        String info = "{\"city\":\"beijing\",\"location\":\"jiuxianqiao__" + time + "\",\"skills\":[\"Java\",\"Golang\",\"Python\"]}";
        ObjectMapper mapper = new ObjectMapper();
        JsonNode node = mapper.readTree(info);

        UserModel userModel = new UserModel();
        userModel.setName("lucumt");
        userModel.setInfo(node);

        userService.addUser(userModel);

        userId = userModel.getId();
        assertTrue(userId > 0);
    }

    @Order(2)
    @Test
    public void testGetUser() {
        UserModel user = userService.getUser(userId);
        JsonNode info = user.getInfo();
        assertNotNull(info);
        System.out.println(info);
    }

    @Order(3)
    @Test
    public void testDeleteUser() {
        userService.deleteUser(userId);
    }

}
```

## 测试结果

![测试结果](/blog_img/mybatis/insert-json-data-in-mybatis/spring-mybatis-json-test-result.png "测试结果") 



