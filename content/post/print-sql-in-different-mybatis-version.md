+++
author = "飞狐"
categories = ["Java编程"]
tags = ["Java","MyBatis"]
date = "2017-12-18T18:33:14+08:00"
description = "在不同版本的mybatis中通过log4j打印实际执行的SQL"
keywords = ["Java","MyBatis"]
title = "在不同版本的MyBatis中通过Log4j打印实际执行的SQL"

+++
项目中ORM框架用的是 **[MyBatis](http://www.mybatis.org/mybatis-3/)**，最近由于业务上的需求将 *MyBatis* 从3.1.1升级到3.4.5，发现升级后通过 **[Log4j](https://logging.apache.org/log4j/1.2/download.html)** 显示SQL的配置方式发生了变化，由于变化较大，故先记录下。  

<!--more-->
假设我们测试的sql文件为 *UserMapper.xml* ， 对应的代码如下，其命名空间为 *com.lucumt.mapper.UserMappper*

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

   <mapper namespace="com.lucumt.mapper.UserMappper">
    	<select id="getUsers" parameterType="String" resultType="com.lucumt.model.UserModel">
        	SELECT id,username,password,create_time AS createTime FROM system_users WHERE username!=#{username}
    	</select>
   </mapper>
```
    
对应的执行代码如下
```java
@Test
public void testMybatis(){
    String resource = "mybatis-config.xml";
    InputStream is = AppTest.class.getClassLoader().getResourceAsStream(resource);
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
    
    SqlSession session = sessionFactory.openSession();
    String statement = "com.lucumt.mapper.UserMappper.getUsers";
    List<UserModel> userList = session.selectList(statement, "admin");
    for(UserModel u:userList){
    	System.out.println(u.toString());
    }
}
```
本文会基于上述代码说明不同版本下如何利用 *Log4j* 在 *MyBatis* 中配置打印日志以及其实现原理。

## MyBatis3.1.1显示SQL的配置与分析

### Log4j相关配置
在 *MyBatis3.1.1* 及以前的版本中若我们想通过 *Log4j* 配置来打印实际执行的SQL，*log4j.properties* 的配置通常类似如下
```java
#在不开启log4j DEBUG模式下显示mybatis中运行的SQL语句 
log4j.logger.java.sql.Connection=DEBUG 
log4j.logger.java.sql.Statement=DEBUG 
log4j.logger.java.sql.PreparedStatement=DEBUG 
log4j.logger.java.sql.ResultSet=DEBUG 
```
### 原理分析 

以 *log4j.logger.java.sql.Connection=DEBUG*  这个配置为例，分析源码可知其sql日志来源于`ConnectionLogger`，查看 *ConnectionLogger* 的代码可知，*ConnectionLogger* 以硬编码的方式生成了一个log对象,当 *DEBUG* 模式开启时该log对象会打印sql语句等信息。
```java
public final class ConnectionLogger extends BaseJdbcLogger implements InvocationHandler {

  //生成一个Connection的log
  private static final Log log = LogFactory.getLog(Connection.class);

  private Connection connection;

  private ConnectionLogger(Connection conn, Log statementLog) {
    super(statementLog);
    this.connection = conn;
    if (isDebugEnabled()) {
      debug("ooo Using Connection [" + conn + "]");
    }
  }

  public Object invoke(Object proxy, Method method, Object[] params)
      throws Throwable {
    try {
      if ("prepareStatement".equals(method.getName())) {
        if (isDebugEnabled()) {//打印执行的SQL语句
          debug("==>  Preparing: " + removeBreakingWhitespace((String) params[0]));
        }        
        PreparedStatement stmt = (PreparedStatement) method.invoke(connection, params);
        stmt = PreparedStatementLogger.newInstance(stmt, getStatementLog());
        return stmt;
      }
      //... other code
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
  }

  //... other code

}
```
运行结果如下  
!["MyBatis3.1.1时显示执行SQL"](/blog_img/print-sql-in-different-mybatis-version/mybatis-3.1.1-print-sql-result.png "MyBatis1.1.1时显示执行SQL")      
从上述代码可知在 *Mybatis3.1.1* 中通过 *Log4j* 实现打印执行SQL的操作很简单，实现原理也易懂，但其存在的一个缺点: **当开启打印SQL日志后，会打印所有正在执行的SQL语句，不能实现针对特定SQL的打印** ，基于此 *MyBatis* 从3.2.0版本之后重新实现了相关功能。

## MyBatis3.4.5显示SQL的配置与分析

### Log4j相关配置
在 *MyBatis3.2.0* 及以后的版本中若我们想通过Log4j配置来打印实际执行的SQL，*log4j.properties* 的配置通常类似如下
```java
#在不开启log4j DEBUG模式下显示mybatis中运行的SQL语句 
log4j.logger.com.lucumt.mapper=DEBUG 
```
在本文写作时，mybatis官网上已有关于这方面更 **[详细的说明](http://www.mybatis.org/mybatis-3/zh/logging.html)** 。

### 原理分析 
同样以 *log4j.logger.java.sql.Connection=DEBUG* 为例，其sql日志来源于 *ConnectionLogger* ，对应代码如下

```java
public final class ConnectionLogger extends BaseJdbcLogger implements InvocationHandler {

  private final Connection connection;

  //通过注入的方式生成log对象
  private ConnectionLogger(Connection conn, Log statementLog, int queryStack) {
    super(statementLog, queryStack);
    this.connection = conn;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] params)
      throws Throwable {
    try {
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, params);
      }    
      if ("prepareStatement".equals(method.getName())) {
        if (isDebugEnabled()) {
          debug(" Preparing: " + removeBreakingWhitespace((String) params[0]), true);
        }        
      }
      //... other code
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
  }

  //... other code

}
```
从上述代码可知，其日志生成是调用`BaseJdbcLogger`的构造方法生成的，*BaseJdbcLogger* 代码如下
```java
public abstract class BaseJdbcLogger {

  protected Log statementLog;
  protected int queryStack;


  public BaseJdbcLogger(Log log, int queryStack) {
    this.statementLog = log;
    if (queryStack == 0) {
      this.queryStack = 1;
    } else {
      this.queryStack = queryStack;
    }
  }
   
  //... other code
}
```
DEBUG模式下查看 *ConnectionLogger* 的调用堆栈如下  
!["ConnectionLogger的调用堆栈"](/blog_img/print-sql-in-different-mybatis-version/connection_logger_stack.png "ConnectionLogger的调用堆栈")  
从其调用堆栈可知log对象是通过`MappedStatement`生成的，如下
```java
public class SimpleExecutor extends BaseExecutor {
   
  //... other code

  @Override
  public <E> List<E> doQuery(MappedStatement ms,Object parameter,
                  RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
      //log对象通过MappedStatement生成
      stmt = prepareStatement(handler, ms.getStatementLog());
      return handler.<E>query(stmt, resultHandler);
    } finally {
      closeStatement(stmt);
    }
  }
}
```
查看 *MappedStatement* 的源码，发现log的生成是在 *Builder* 方法中，如下
```java
public final class MappedStatement {

  public static class Builder {
    private MappedStatement mappedStatement = new MappedStatement();

    public Builder(Configuration configuration, String id, SqlSource sqlSource, SqlCommandType sqlCommandType) {
      mappedStatement.configuration = configuration;
      mappedStatement.id = id;
      mappedStatement.sqlSource = sqlSource;
      mappedStatement.statementType = StatementType.PREPARED;
      mappedStatement.parameterMap = new ParameterMap.Builder(configuration, "defaultParameterMap", null, new ArrayList<ParameterMapping>()).build();
      mappedStatement.resultMaps = new ArrayList<ResultMap>();
      mappedStatement.sqlCommandType = sqlCommandType;
      mappedStatement.keyGenerator = configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType) ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
      String logId = id;
      //可以通过设置logPrefix的方法来生成log对象
      if (configuration.getLogPrefix() != null) {
        logId = configuration.getLogPrefix() + id;
      }
      //通过logId生成log对象
      mappedStatement.statementLog = LogFactory.getLog(logId);
      mappedStatement.lang = configuration.getDefaultScriptingLanguageInstance();
    }
}
```
通过上面的代码可知log对象是由logId生成的，进一步debug发现logId是由 **namespace+方法id** 组成，在本例中为 *com.lucumt.mapper.UserMappper.getUsers* ，而前面的配置为 *log4j.logger.com.lucumt.mapper=DEBUG* ，由于 *Log4j* 中的log示例的继承关系，相当于 *com.lucumt.mapper.UserMappper.getUser* 也开启了DEBUG模式，故在实际执行时可以显示打印SQL语句，运行结果如下  
!["MyBatis3.4.5时显示执行SQL"](/blog_img/print-sql-in-different-mybatis-version/mybatis-3.4.5-print-sql-result.png "MyBatis3.4.5时显示执行SQL")  
利用新版 *MyBatis* 的这一特性，我们可以实现类似如下的不同粒度sql打印

```java
log4j.logger.com.xxx.mapper=DEBUG #打印xxx包下所有的执行SQL
log4j.logger.com.yyy.mapper.PersonMapper=DEBUG #打印PersonMapper下所有的执行SQL
log4j.logger.com.zzz.mapper.GroupMapper.getGroups=DEBUG #只打印getGroups对应的执行SQL  
```

<br/>
由前面的代码可知 *MappedStatement* 的 *Build* 方法在生成log对象时会检测是否有 *logPrefix* 配置，若有则用 *logPrefix* 来生成log对象，于是可以通过设置 *logPrefix* 以另外一种方式配置打印sql。 可在 *MyBatis* 配置文件中添加如下配置 

```xml
<settings>
   <setting name="logPrefix" value="dao."/> <!-- 设置前缀为dao -->
   <setting name="logImpl" value="log4j"/> <!-- 设置使用log4j为日志实现类 -->
</settings>
```
然后将 *log4j.properties* 的配置修改为

```ruby
log4j.logger.dao=DEBUG
```
执行结果与前面相同，通过 *logPrefix* 可以在有些时候简化sql打印配置。

### 待分析问题
若将 *MyBatis* 的版本变 *3.3.0* 时，通过 *Log4j* 配置打印SQL时，如下所示的配置方式只有部分生效，原因待分析

```java
log4j.logger.com.xxx=DEBUG #可以打印SQL
log4j.logger.com.xxx.mapper=DEBUG #可以打印SQL
log4j.logger.com.xxx.mapper.UserMapper=DEBUG #不能打印SQL
log4j.logger.com.xxx.mapper.UserMapper.getUsers=DEBUG #不能打印SQL
```