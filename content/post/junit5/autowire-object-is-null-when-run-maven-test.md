---
title: "利用SpringBoot3与JUnit5进行单元测试时依赖注入对象报空指针"
date: 2025-03-24T10:18:30+08:00
lastmod: 2025-03-24T10:18:30+08:00
draft: false
keywords: ["junit5","springboot",“空指针”]
description: "简单记录在利用SpringBoot3与JUnit5进行单元测试时依赖注入对象报空指针的问题，以及对应的解决方案"
tags: ["junit5","springboot"]
categories: ["单元测试"]
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
codeTabSeperator: "::"
moreMeta: true

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

近期有个项目采用了[**SpringBoot3** ](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes)以及[**JUnit5**](https://junit.org/junit5)进行单元测试，在此过程中一直提示`Spring`中依赖注入报错的问题，该问题的解决方案比较特殊，主要是升级[**maven-surefire-plugin**](https://maven.apache.org/surefire/maven-surefire-plugin)的版本，故简单记录下。

<!--more-->

## 问题描述

相关代码如下，主要基于`UserMapperTest`进行测试

{{% codetabs %}}  

```java::测试类代码
import com.lucumt.app.AuthApplication;
import com.lucumt.app.model.SysUser;
import jakarta.annotation.Resource;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

import static org.junit.jupiter.api.Assertions.assertFalse;

@Slf4j
@SpringBootTest(classes = AuthApplication.class)
public class UserMapperTest {

    @Resource
    private SysUserMapper userMapper;

    @BeforeAll
    public static void init(){
        System.setProperty("nacos.logging.default.config.enabled", "false");
    }

    @Test
    public void testSelectUser(){
        List<SysUser> userList = userMapper.selectList(null);
        userList.forEach(s->log.info(s.toString()));
        assertFalse(userList.isEmpty());
    }
}
```

```java::接口类代码
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.lucumt.app.model.SysUser;

public interface SysUserMapper extends BaseMapper<SysUser> {
}
```

```java::启动类代码
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
@MapperScan("com.lucumt.app.mapper")
@SuppressWarnings("uncommentedmain")
public class AuthApplication {

    public static void main(String[] args) {
        System.setProperty("nacos.logging.default.config.enabled", "false");
        SpringApplication.run(AuthApplication.class);
    }
}
```

```xml::父pom文件
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lucumt.app</groupId>
    <artifactId>auth</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <description>应用中心 - 权限与认证模块</description>
    <modules>
        <module>app</module>
        <module>common</module>
    </modules>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring.boot.version>3.2.12</spring.boot.version>
        <spring.cloud.version>2023.0.1</spring.cloud.version>
        <nacos.version>2023.0.1.0</nacos.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.15.4</version>
        </dependency>
        <dependency>
            <groupId>org.json</groupId>
            <artifactId>json</artifactId>
            <version>20220320</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.36</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                    <encoding>${project.encoding}</encoding>
                </configuration>
            </plugin>
            <!-- 持续集成时检查代码规范 -->
            <plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
                <version>3.6.0.1398</version>
            </plugin>
            <!-- 检查代码规范 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>3.1.2</version>
                <dependencies>
                    <dependency>
                        <groupId>com.puppycrawl.tools</groupId>
                        <artifactId>checkstyle</artifactId>
                        <version>8.41.1</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <configLocation>style/checkStyle.xml</configLocation>
                    <encoding>UTF-8</encoding>
                    <consoleOutput>true</consoleOutput>
                    <failsOnError>true</failsOnError>
                    <linkXRef>false</linkXRef>
                </configuration>
                <executions>
                    <execution>
                        <id>validate</id>
                        <phase>validate</phase>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!-- 上传jar包到仓库 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.8.2</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.4.3</version>
                <configuration>
                    <encoding>utf-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>release</id>
            <url>https://mirrors.lucumt.com/repository/Maven-local</url>
        </repository>
    </repositories>

    <distributionManagement>
        <repository>
            <id>release</id>
            <url>https://mirrors.lucumt.com/repository/Maven-local</url>
        </repository>
    </distributionManagement>

</project>
```

```xml::模块pom文件
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.lucumt.app</groupId>
        <artifactId>auth</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>auth-app</artifactId>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <nacos.version>2023.0.1.0</nacos.version>
        <mysql.version>8.0.15</mysql.version>
        <mybatis-plus.version>3.5.5</mybatis-plus.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.lucumt.app</groupId>
            <artifactId>auth-common</artifactId>
            <version>1.0-SNAPSHOT</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.vaadin.external.google</groupId>
                    <artifactId>android-json</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
            <version>${nacos.version}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>${nacos.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring.boot.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

{{% /codetabs %}}  

在`Intellij Idea`中直接运行该测试类，可运行成功

![IDEA中直接运行成功](/blog_img/junit/autowire-object-is-null-when-run-maven-test/run-junit-test-directly-in-idea.png "IDEA中直接运行成功")   

在控制台执行`mvn clean compile test`时，报如下所示的空指针错误

![命令行中通过maven运行报错](/blog_img/junit/autowire-object-is-null-when-run-maven-test/run-junit-test-with-maven-error.png "命令行中通过maven运行报错")   

## 解决方案

一开始自己主要通过`Google`搜索，主要找到[回复1](https://stackoverflow.com/questions/62871584/why-is-an-autowired-controller-always-null-in-junit5-tests)和[回复2](https://stackoverflow.com/questions/54928869/migrating-from-junit4-to-junit5-throws-nullpointerexception-on-autowired-reposi)，但均未能解决报错问题。

咨询`Deepseek`后，其给出的解决方案如下，挨个对照检查，自己的项目中并没有违反相关约束。

| 错误原因                   | 解决方案                               |
| :------------------------- | :------------------------------------- |
| 缺少 `@SpringBootTest`     | 添加 `@SpringBootTest` 注解            |
| 包路径不正确               | 调整测试类包位置或显式指定扫描路径     |
| 依赖未正确引入             | 检查 `spring-boot-starter-test` 依赖   |
| Bean未被`Spring`管理       | 添加 `@Component` 等注解或检查扫描配置 |
| 混淆`JUnit4`/`JUnit5` 注解 | 移除 `@RunWith`，使用`JUnit5`注解      |

通过添加`MyBatis`相关的说明，对提示词进行细化后，回答如下，挨个对照检查，依旧不能解决问题。

| **现象**                                       | **解决方案**                             |
| :--------------------------------------------- | :--------------------------------------- |
| `Mapper`接口未被扫描到                         | 添加 `@MapperScan` 或检查包路径          |
| 数据源配置错误导致`MyBatis`初始化失败          | 检查测试环境的 `application.properties`  |
| `XML`映射文件未找到                            | 确认 `mybatis.mapper-locations` 配置正确 |
| `MyBatis`依赖版本不兼容                        | 升级至与`Spring Boot 3.2`兼容的版本      |
| 测试未使用 `@Transactional` 导致数据库回滚问题 | 添加 `@Transactional` 注解在测试类或方法 |

无奈之下只能继续搜索，找到了印度三哥写的[这篇文章](https://mkyong.com/maven/maven-test-failed-on-spring-autowired-and-junit-5)，其中说明`maven-surefire-plugin`的版本太旧，需要进行升级。

去对应报错信息处查看，确实如此

![maven-surefire-plugin插件版本](/blog_img/junit/autowire-object-is-null-when-run-maven-test/maven-surefire-plugin-version.png "maven-surefire-plugin插件版本")   

在模块pom文件中添加如下高亮部分的配置

```xml{data-line="3-7"}
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.0</version>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>${spring.boot.version}</version>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

重新执行`mvn clean compile test`，可看出能正常执行成功。

![命令行中通过maven运行成功](/blog_img/junit/autowire-object-is-null-when-run-maven-test/run-junit-test-with-maven-success.png "命令行中通过maven运行成功")   