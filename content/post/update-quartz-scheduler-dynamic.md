+++
author = "飞狐"
categories = ["Web编程","Java编程"]
tags = ["Quartz","Spring","SpringMVC"]
date = "2018-01-09T22:10:30+08:00"
description = "Blog of Rosen Lu"
keywords = ["Quartz","Spring","SpringMVC"]
title = "在Quartz中动态设置定时任务的执行时间"

+++

**[Quartz](http://www.quartz-scheduler.org/)**是软件开发中常用的任务调度框架，实际中通常结合 **[Spring](https://spring.io/)** 一起使用，并在 *Spring* 的配置文件中利用 *0 0 12 ? \* WED* 这种方式以硬编码的方式配置定时任务的执行时间。有时候需要动态的设置定时任务的执行时间，如让用户自己选择何时备份数据，此时就需要采用动态设置其执行时间。

<!--more-->
为实现动态设置定时任务执行时间的功能，首先需要实现以硬编码的方式设置定时任务执行时间，然后在其基础上修改为可动态设置，本文基于这两分部分逐步介绍如何实现。

## 硬编码设置定时时间
本文采用 Quartz + **[SpringMVC](https://spring.io/guides/gs/serving-web-content/)** 的实现框架，同时基于 **[Maven](https://maven.apache.org/)**运行，相关配置过程如下：   
1.首先在 *POM* 文件中引入相应的依赖JAR包。  
```xml
<dependencies>
	<dependency>
		<groupId>org.quartz-scheduler</groupId>
		<artifactId>quartz</artifactId>
		<version>2.3.0</version>
	</dependency>
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-api</artifactId>
		<version>1.7.25</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context</artifactId>
		<version>4.3.13.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context-support</artifactId>
		<version>4.3.13.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-web</artifactId>
		<version>4.3.13.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-webmvc</artifactId>
		<version>4.3.13.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-tx</artifactId>
		<version>4.3.13.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.9</version>
		<scope>test</scope>
	</dependency>
	<dependency>
		<groupId>javaee</groupId>
		<artifactId>javaee-api</artifactId>
		<version>5</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
```

2.创建一个定时任务测试类 *TestJob*  
	
```java
public class TestJob {

	private DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  

	public void schedulerJob(){
		System.out.println("=========定时输出:\t"+df.format(new Date()));
	}
}
```

3.结合 *Spring* 进行定时任务的配置    

```xml
<beans xmlns="http://www.springframework.org/schema/beans" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-4.3.xsd ">

	<bean id="testJob" class="com.lucumt.quartz.TestJob"></bean>
	<bean id="testJobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
		<!-- 指定任务类 -->
		<property name="targetObject" ref="testJob" />
		<!-- 指定任务执行的方法 -->
		<property name="targetMethod" value="schedulerJob" />
	</bean>
	<bean id="testJobTrigger"
		class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
		<property name="jobDetail" ref="testJobDetail" />
		<!-- 每10秒运行一次 -->
		<property name="cronExpression" value="0/10 * * * * ?" />
	</bean>

	<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
		<property name="triggers">
			<list>
				<ref bean="testJobTrigger" />
			</list>
		</property>
	</bean>
</beans>
```

4.*web.xml* 中配置如下：

```xml
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee  http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	<display-name>Dynamic Quartz Scheduler</display-name>
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath*:spring-context-*.xml</param-value>
	</context-param>

</web-app>
```
<br/>
配置完成后，在 *eclipse* 中运行`tomcat7:run`运行结果如下，可以看出定时任务每隔10秒执行一次。    
!["未修改之前的定时任务输出"](/blog_img/update-quartz-scheduler-dynamic/static_scheduler_output.png "未修改之前的定时任务输出")  
上述的硬编码设置将 *Quartz* 的执行时间通过硬编码方式写入XML配置文件中，这是最常见的用法，但通过XML配置文件写入定时时间时无法动态的更改其执行时间。

## 动态设置定时时间
为了便于演示，本文采用Web程序的方式展示相关操作过程。  
1.增加一个 *testScheduler.jsp* 展示操作界面:  
```html
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="content-type" content="text/html; charset=UTF-8">
	<title>动态设置quartz</title>
	<script type="text/javascript" src="js/jquery-2.1.1.min.js"></script>
	<link rel="stylesheet" type="text/css" href="js/bootstrap/css/bootstrap.min.css"/>
	<script type="text/javascript" src="js/bootstrap/js/bootstrap.min.js"></script>
	<style type="text/css">
	  .container{
	     margin-top: 30px;
	     margin-left: auto;
	     margin-right: auto;
	     padding: 10px;
	     background-color: #d0d0d0;
	     border-radius: 5px;
	     min-height:400px;
	  }
	  
	  .hidden{
	     display: none;
	  }
	</style>
	<script type="text/javascript">
	  function changeScheduler(){
		  var hiddenId = $(".hidden").attr("id");
		  var expression = null;
		  if(hiddenId=="scheduler_one"){
			  $("#scheduler_one").removeClass("hidden");
			  $("#scheduler_two").addClass("hidden");
			  expression="0/10 * * * * ?";
		  }else{
			  $("#scheduler_one").addClass("hidden");
			  $("#scheduler_two").removeClass("hidden");
			  expression="0/30 * * * * ?";
		  }
		  sendChangeRequest(expression);
	  }
	  
	  function sendChangeRequest(expression){
		  $.ajax({
			  url:"changeScheduler",
			  type:"post",
			  data:{
				  expression:expression
			  },
			  success:function(){
			  }
		  });
	  }
	</script>
</head>
<body>
<div class="container-fluid container">
     <div id="scheduler_one">
                      当前定时任务的表达式为<b>0/10 * * * * ?</b>,每隔10秒输出一次
     </div>
     <div id="scheduler_two" class="hidden">
                      当前定时任务的表达式为<b>0/30 * * * * ?</b>,每隔30秒输出一次
     </div>
     <button type="button" class="btn btn-primary btn-sm" onclick="changeScheduler()">切换定时时间</button>
</div>
</body>
</html>
```
2.增加一个Controller类 *QuartzController* 用于响应前端重新设置定时任务时间的请求  
```java
@Controller("/")
public class QuartzController {
	
	@Autowired
	private JobScheduler jobScheduler;

	@RequestMapping("testScheduler")
	public String testScheduler(){
		return "testScheduler";
	}
	
	@RequestMapping("changeScheduler")
	@ResponseBody
	public String changeScheduler(String expression){
		System.out.println("执行时间被修改为:\t"+expression);
		jobScheduler.resetJob(expression);
		return "SUCCESS";
	}

}
```
3.在时任务测试类 *TestJob* 中添加一个 *resetJob* 方法，用于重新设置定时任务执行时间  

```java
public class JobScheduler implements ServletContextAware {

	private DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	
	private ServletContext context;
	
	@Override
	public void setServletContext(ServletContext context) {
       this.context=context;		
	}

	public void schedulerJob() {
		System.out.println("=========定时输出:\t" + df.format(new Date())); 
	}
	
    //通过此方法重新设置定时任务调度时间
	public void resetJob(String expression){
		ApplicationContext applicationContext = WebApplicationContextUtils.getRequiredWebApplicationContext(context);
		Scheduler scheduler = (Scheduler) applicationContext.getBean("testScheduler");
		CronTriggerImpl trigger = null;
		try {
			TriggerKey triggerKeys = TriggerKey.triggerKey("testJobTrigger",Scheduler.DEFAULT_GROUP);
			trigger = new CronTriggerImpl();
			trigger.setCronExpression(expression);
			trigger.setKey(triggerKeys);//要确保key相同
			scheduler.rescheduleJob(triggerKeys,trigger);
		} catch (ParseException | SchedulerException e) {
			e.printStackTrace();
		}
	}
}
```  

4.其他配置文件保持不变，修改后的运行界面如下  
!["修改定时任务的操作界面"](/blog_img/update-quartz-scheduler-dynamic/dynamic_scheduler_button.png "修改定时任务的操作界面")  
5.多次点击该按钮，控制台输出如下，可以看出实现了动态设置定时任务的功能  
!["动态修改定时任务后的运行效果"](/blog_img/update-quartz-scheduler-dynamic/dynamic_scheduler_output.png "动态修改定时任务后的运行效果")  

上述代码是基于 *Quartz2.3.0* 来实现的，相关源代码请参见 **[quartz_demo](https://github.com/lucumt/myrepository/tree/master/java/quartz_demo)** ，其核心在于 *resetJob* 方法通过调用 **[CronTriggerImpl](https://github.com/quartz-scheduler/quartz/blob/master/quartz-core/src/main/java/org/quartz/impl/triggers/CronTriggerImpl.java)** 来重新设置定时任务执行时间，需要注意的是要确保定时任务修改前后的 *triggerKey* 一致，这样才能修改生效，否则应用程序会在执行原有的定时任务时同时以新的时间来执行新的定时任务，即同时执行两个定时任务，达不到预期效果。

## Quartz1.7.2中的定时任务设置
在旧版的 *Quartz(1.7.2)* 中 *rescheduleJob* 的方法参数发生了变化，相应的 *Spring* 版本也发生了变化，需要用 **[CronTriggerBean](https://docs.spring.io/spring/docs/3.0.x/javadoc-api/org/springframework/scheduling/quartz/CronTriggerBean.html)** 替换 *CronTriggerImpl*，对应的实现代码可修改为如下：  

```java
public void resetJob(String expression){
	ApplicationContext applicationContext = WebApplicationContextUtils.getRequiredWebApplicationContext(context);
	Scheduler scheduler = (Scheduler) applicationContext.getBean("testScheduler");
	try {
		CronTriggerBean trigger = new CronTriggerBean();
		trigger.setCronExpression(expression);
		trigger.setName("testJobTrigger");
		trigger.setGroup(Scheduler.DEFAULT_GROUP);
		trigger.setJobName("testJobDetail");
		scheduler.rescheduleJob("testJobTrigger", Scheduler.DEFAULT_GROUP, trigger);
	} catch (SchedulerException | ParseException e) {
		e.printStackTrace();
	}
}
```
其运行结果和前面的一致。

## 通过Spring获取Trigger导致的重复执行问题
将上述代码中的 *CronTriggerBean* 初始化从 *new* 关键字实现变为通过 *Schduler* 获取原有的任务后重新更新，修改后的代码如下：

```java
public void resetJob(String expression){
    ApplicationContext applicationContext = WebApplicationContextUtils.getRequiredWebApplicationContext(context);
    Scheduler scheduler = (Scheduler) applicationContext.getBean("testScheduler");
    try {
        CronTriggerBean trigger = (CronTriggerBean) scheduler.getTrigger("testJobTrigger", Scheduler.DEFAULT_GROUP);//通过scheduler获取
        trigger.setCronExpression(expression);
        trigger.setName("testJobTrigger");
        scheduler.rescheduleJob("testJobTrigger", Scheduler.DEFAULT_GROUP, trigger);
    } catch (SchedulerException | ParseException e) {
        e.printStackTrace();
    }
}
```
实际运行时会发现每次动态切换 *Quartz* 的执行时间时都会导致该定时任务被执行两次或错误执行的现象，如下图所示：  
!["动态修改定时任务后定时任务错误执行"](/blog_img/update-quartz-scheduler-dynamic/dynamic_scheduler_output_wrong.png "动态修改定时任务后定时任务错误执行")  
初步上述问题产生的原因为通过 *Scheduler* 获取的是已有的 *Trigger* 而导致重复执行（不论 *Quartz* 新旧版本均有此问题 ），如果采用 *new* 关键字重新创建一个 *Trigger* 则此问题会消失，至于为何采用旧的 *Trigger* 会导致定时任务错误执行，还有待进一步分析。