+++

author = "飞狐"
categories = ["Java编程","Spring系列","单元测试"]
tags = ["Java","JUnit"]
date = "2016-03-20T16:27:21+08:00"
description = "本文主要介绍了基于Spring来进行JUnit单元测试时如何对数据库事务进行控制"
keywords = ["Java","JUnit"]
title = "利用Spring和JUnit对数据库操作进行单元测试"

+++

在进行Java程序开发时，我们偶尔会被要求使用**[JUnit](http://junit.org/)**进行单元测试来确保我们所写的程序逻辑是正确的。一个良好的单元测试应该具备 ***覆盖度高，可重复执行，单一性*** 等特点。本文主要关注***可重复执行*** ，在Web开发中，大部分方法都会使数据库的记录发生变化，为了能够重复执行，必须利用**[数据库事务](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1)** 来进行 ***回滚*** 从而达到重复执行的目的。最原始的方法是利用 **[java.sql.Connection](https://docs.oracle.com/javase/7/docs/api/java/sql/Connection.html)** 类的 `commit()` 或 `rollback()` 方法来在每个单元测试方法中手动的进行提交或回滚，此种方式使得单元测试代码嵌入了与实际业务逻辑无关的数据库操作事务控制代码。利用**[Spring](https://spring.io/)**和**[JUnit](http://junit.org/)**通过注解的方式我们可以很容易的对单元测试中的数据库操作进行事务控制。
<!--more-->

## 所有方法都回滚
在该单元测试类的开头加上 `@TransactionConfiguration(defaultRollback=true)` 可以确保该类中的所有方法在执行完毕之后默认都进行回滚。
```java
package com.hirain.testmanagement.service.test;
 
import static org.junit.Assert.assertEquals;
 
import java.util.Date;
 
import javax.inject.Inject;
 
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.transaction.TransactionConfiguration;
import org.springframework.transaction.annotation.Transactional;
 
import com.hirain.testmanagement.common.util.StringUtil;
import com.hirain.testmanagement.model.ProjectModel;
import com.hirain.testmanagement.service.IProjectService;
 
@RunWith(SpringJUnit4ClassRunner.class)
@Transactional
@TransactionConfiguration(defaultRollback=true)
@ContextConfiguration("classpath:spring/spring-context-*.xml")
public class ProjectServiceTest{
 
	@Inject
	private IProjectService projectService;
	   
	@Test
	@Transactional
	public void testAddProject(){
	  ProjectModel pModel=new ProjectModel();
	  String projectId=StringUtil.getUUID();
	  pModel.setId(projectId);
	  pModel.setName("汽车电子测试管理系统");
	  pModel.setAlias("INTA");
	  pModel.setLastModifyTime(new Date());
	  pModel.setLastModifyUser("6e518d0819d14148ae489f76dad80967");
	  pModel.setCreateTime(new Date());
	  pModel.setCreateUser("cface18d5fac11e28c68c89cdca4c015");
	  projectService.addProject(pModel);
	  assertEquals("Add project failed!",projectService.getProject(projectId).getName(),pModel.getName());
	}
 
}
```  

## 指定方法回滚
若想只对某个特定的方法进行回滚，需要在该单元测试类的开头去掉 `@TransactionConfiguration(defaultRollback=true)` ，同时在对应的方法上加上注解声明 `@Rollback(true)` 即可达到目的。
```java
package com.hirain.testmanagement.service.test;
 
import static org.junit.Assert.assertEquals;
 
import java.util.Date;
 
import javax.inject.Inject;
 
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;
 
import com.hirain.testmanagement.common.util.StringUtil;
import com.hirain.testmanagement.model.ProjectModel;
import com.hirain.testmanagement.service.IProjectService;
 
@RunWith(SpringJUnit4ClassRunner.class)
@Transactional
@ContextConfiguration("classpath:spring/spring-context-*.xml")
public class ProjectServiceTest{
 
	@Inject
	private IProjectService projectService;
	   
	@Test
	@Rollback(true)
	public void testAddProject(){
	  ProjectModel pModel=new ProjectModel();
	  String projectId=StringUtil.getUUID();
	  pModel.setId(projectId);
	  pModel.setName("汽车电子测试管理系统");
	  pModel.setAlias("INTA");
	  pModel.setLastModifyTime(new Date());
	  pModel.setLastModifyUser("6e518d0819d14148ae489f76dad80967");
	  pModel.setCreateTime(new Date());
	  pModel.setCreateUser("cface18d5fac11e28c68c89cdca4c015");
	  projectService.addProject(pModel);
	  assertEquals("Add project failed!",projectService.getProject(projectId).getName(),pModel.getName());
	}
 
}
```
 
 