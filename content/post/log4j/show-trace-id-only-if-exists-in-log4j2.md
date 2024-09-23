---
title: "在Log4j2中实现只有在traceId有值时才输出"
date: 2022-10-13T17:47:46+08:00
lastmod: 2022-10-13T17:47:46+08:00
draft: false
keywords: ["log4j2","traceId","链路追踪"]
description: "简要介绍如何实现在Log4j2中实现只有在traceId有值时才输出，以便增强微服务开发模式下的日志追踪与分析"
tags: ["java","log4j"]
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

记录如何在[**Log4j2**](https://logging.apache.org/log4j/2.x/)中使用traceId实现[**链路追踪**](https://blog.51cto.com/u_15501087/5834195)时只有在请求header中有`trace-id`时才显示前端发送过来的`trace-id`值。

<!--more-->

## 前置工作

完整代码参见[**spring-mybatis-demo**](https://github.com/fox-world/spring-mybatis-demo.git)。

假设使用的程序框架为`Spring Boot`并通过`Maven`管理依赖，需要预先添加如下代码配置：

1. `pom.xml`中排除`Spring Boot`自带的日志框架并添加`log4j2`依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter</artifactId>
       <version>${spring.boot.version}</version>
       <exclusions>
           <exclusion>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-logging</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
       <version>${spring.boot.version}</version>
       <exclusions>
           <exclusion>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-logging</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-log4j2</artifactId>
       <version>${spring.boot.version}</version>
   </dependency>
   ```

2. 添加`Filter`用于从请求header中获取前端传递的`trace-id`并传入[**MDC**](https://logback.qos.ch/manual/mdc.html)中

   ```java
   @Order(0)
   @Component
   @WebFilter(filterName = "traceIdFilter", urlPatterns = "/*")
   public class TraceIdFilter implements Filter {
   
       public static final String TRACE_ID = "TRACE_ID";
       public static final String TRACE_ID_HEADER = "trace-id";
   
       @Override
       public void init(FilterConfig filterConfig) throws ServletException {
       }
   
       @Override
       public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws ServletException, IOException {
           HttpServletRequest request = (HttpServletRequest) req;
           String traceId = request.getHeader(TRACE_ID_HEADER);
           MDC.put(TRACE_ID, traceId);
           chain.doFilter(req, res);
       }
   
       @Override
       public void destroy() {
           MDC.clear();
       }
   
   }
   ```

3. 在主启动类中添加如下配置用于在线程间传递`MDC`值

   ```java
   @SpringBootApplication
   public class TestApplication {
   
       // 用于在线程间传递mdc值
       static {
           System.setProperty("log4j2.isThreadContextMapInheritable", "true");
       }
   
       public static void main(String[] args) {
           SpringApplication.run(TestApplication.class, args);
       }
   }
   ```

4. `log4j2.xml`配置如下

   ```xml
   <Configuration status="warn">
       <Appenders>
           <!-- Console appender configuration -->
           <Console name="console" target="SYSTEM_OUT">
               <PatternLayout pattern="%yellow{[%d{yyyy-MM-dd HH:mm:ss.SSS}]} %clr{[%-5level]} %green{[TraceId-%X{TRACE_ID}]} %cyan{[%c]}- %msg%n" />
           </Console>
       </Appenders>
       <Loggers>
           <!-- Root logger referring to console appender -->
           <Root level="info" additivity="false">
               <AppenderRef ref="console" />
           </Root>
       </Loggers>
   </Configuration>
   ```

   其中的`%green{[TraceId-%X{TRACE_ID}]}`就是将获取到的`trace-id`在控制台中按照`TraceId-xxx`的格式以绿色打印出来，该段配置是我们进行链路最终的核心。

   其中`%X`专门用于获取`MDC`中的值，在`log4j2`官网对应的说明如下

   ![log4j2关于%X的用法说明](/blog_img/log4j/show-trace-id-only-if-exists-in-log4j2/log4j2-%X-instruction.png "log4j2关于%X的用法说明") 

## 测试&问题

测试代码如下

```java
@Slf4j
@RestController
@RequestMapping("user")
public class UserController {

    @Resource
    private UserService userService;

    @RequestMapping("queryAll")
    public List<UserModel> all() {
        List<UserModel> userList = userService.queryAllUsers();
        log.info("==========all user size: {}", userList.size());
        return userList;
    }

}
```

在Postman中发送类似如下请求，在header中将`trace-id`的值设置为123456

![通过postman发送带header的请求](/blog_img/log4j/show-trace-id-only-if-exists-in-log4j2/postman-test-interface.png "通过postman发送带header的请求") 

在控制台可发现类似如下输出，可发现前面几条系统启动时输出的日志也会按照`trace-id`格式进行输出，由于此时还没接收到前端发送的请求，故其值为空导致只显示一个`TraceId-`前缀，**看起来不美观且影响使用**。

![traceId没有值时也会输出](/blog_img/log4j/show-trace-id-only-if-exists-in-log4j2/log4j2-with-empty-trace-id-output.png "traceId没有值时也会输出") 

## 解决方案

终极目标是实现在有`trace-id`时按照格式输出，没有时则正常输出，很明显要达成此效果需要判断`trace-id`是否为空。

最开始`Google`和`Stackoverflow`搜索的结果建议使用[**ScriptPatternSelector**](https://logging.apache.org/log4j/2.x/manual/layouts.html#scriptpatternselector)，在官网给出的使用示例如下，看起来有一丢丢复杂，在有多个[**Appenders**](https://logging.apache.org/log4j/2.x/manual/appenders.html)时不适合扩展。

```xml
<PatternLayout>
  <ScriptPatternSelector defaultPattern="[%-5level] %c{1.} %C{1.}.%M.%L %msg%n">
    <Script name="BeanShellSelector" language="bsh"><![CDATA[
      if (logEvent.getLoggerName().equals("NoLocation")) {
        return "NoLocation";
      } else if (logEvent.getMarker() != null && logEvent.getMarker().isInstanceOf("FLOW")) {
        return "Flow";
      } else {
        return null;
      }]]>
    </Script>
    <PatternMatch key="NoLocation" pattern="[%-5level] %c{1.} %msg%n"/>
    <PatternMatch key="Flow" pattern="[%-5level] %c{1.} ====== %C{1.}.%M:%L %msg ======%n"/>
  </ScriptPatternSelector>
</PatternLayout>
```

上述方式需要设置两个`Pattern`且在不同的`Appender`中都需要动态配置，使用起来不是特别灵活。



有木有可能通过一个`Pattern`同时兼容这两种情况呢？



经过网络搜索和查看`log4j2`中关于[**Pattern Layout**](https://logging.apache.org/log4j/2.x/manual/layouts.html#pattern-layout)的说明之后，最终找到了解决方案，那就是`%notEmpty`!

官方文档中关于`%notEmpty`的说明如下

![log4j2关于%notEmpty的用法说明](/blog_img/log4j/show-trace-id-only-if-exists-in-log4j2/log4j2-%notEmpty-instruction.png "log4j2关于%notEmpty的用法说明") 

基于上述说明可尝试将`%green{%notEmpty{[TraceId-%X{TRACE_ID}]}}`修改为`%green{%notEmpty{[TraceId-%X{TRACE_ID}]}}`，最终完整的`PatternLayout`如下

```xml
<PatternLayout pattern="%yellow{[%d{yyyy-MM-dd HH:mm:ss.SSS}]} %clr{[%-5level]} %green{%notEmpty{[TraceId-%X{TRACE_ID}]}} %cyan{[%c]}- %msg%n" />
```

重新调用`Postman`的输出结果如下，完美实现功能！

![traceId只有在有值时才输出](/blog_img/log4j/show-trace-id-only-if-exists-in-log4j2/log4j2-with-not-empty-trace-id-output.png "traceId只有在有值时才输出") 

## 服务间调用

在微服务中通常会有多个模块协同工作，各个模块之间也可能会互相调用，在此种情况下也需要将`trace-id`准确的传递与获取，确保单次请求调用链路中的`trace-id`都一样才能体现链路追踪的意义。

### Feign调用

当使用`Feign`作为远程调用实现时，需在**被调用方**中添加如下配置代码才能确保`trace-id`被正常传递与获取。

```java
import com.hirain.orienlink.mfs.filter.TraceIdFilter;
import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.slf4j.MDC;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate requestTemplate) {
        String traceId = String.valueOf(MDC.get(TraceIdFilter.TRACE_ID));
        requestTemplate.header(TraceIdFilter.TRACE_ID_HEADER, traceId);
    }
}
```



