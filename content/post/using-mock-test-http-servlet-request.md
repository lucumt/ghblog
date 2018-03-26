+++

author = "飞狐"
categories = ["Java编程","Spring系列","单元测试"]
tags = ["Java","Spring","SpringMVC","JUnit"]
date = "2016-03-19T22:43:47+08:00"
description = "在Spring中利用Mock对HttpServletRequest进行单元测试"
keywords = ["Java","Spring","SpringMVC","JUnit"]
title = "在Spring中利用Mock对HttpServletRequest进行单元测试"

+++

## 编写单元测试时的注意事项
根据软件开发过程中的***[TDD](https://en.wikipedia.org/wiki/Test-driven_development)*** 理论，在我们编写自己的代码时，要尽量使得该代码能够进行单元测试。为了能够使得代码可以进行单元测试，我们在给接口或方法传入参数时要尽量传入简单参数，避免传入 ***HttpServletRequest*** , ***ServletContext*** 等和web上下文相关的复杂对象。但仍有部分情况下基于代码简洁性和可维护性的考虑，我们需要传入 ***HttpServletRequest*** 对象，此时对此类方法进行**[JUnit](http://junit.org/)**单元测试时会较为困难，本文介绍一种在**[Spring](https://spring.io/)**中通过**[Mock](http://mockito.org/)**来模拟***HttpServletRequest*** 对象进行**[JUnit](http://junit.org/)**单元测试的方法。

<!--more-->
 
假设在 ***HttpServletRequest*** 中有一个userId字符串对象，我们想在queryUserById方法中调用该参数来获取用户信息，则正确的做法应如下所示:
```java
String userId = request.getAttribute("userId").toString();//先获取userId对象
queryUserById(userId);//然后将获取的userId传入对应方法

public User queryUserById(String userId){//相关该方法
   User userModel = userDao.findById(userId);
   return userModel;
}  
```
请尽量避免使用第二种方式
```java
queryUserById(request);//直接传入request对象

public User queryUserById(HttpServletRequest request){//相关方法
   String userId = request.getAttribute("userId").toString();//在该方法内部获取userId

   User userModel = userDao.findById(userId);
   return userModel;
}  
```
*若采用第一种方法，我么在进行单元测试时，可以很容易的自己制造一个String字符串来代表userId进行测试，但当采用第二种方法后，在进行单元测试时我们是比较难以模拟一个 **HttpServletRequest** 对象，从而影响我们的测试。*

## Spring和Mock在单元测试中的使用
在某些方法中，为了减少代码量和提高程序的可读性，我们有时候需要直接传入 ***HttpServletRequest*** 或 ***ServletContext*** 对象，如果我们想对这种方法进行测试，可以利用**[Mock](http://mockito.org/)**来模拟相关的对象。
 
由于**[Spring](https://spring.io/)**自身已经整合了**[Mock](http://mockito.org/)**相关的类，故在此处展示一个示例代码，以供参考:
```java
import java.io.File;
 
import org.junit.Test;
import org.springframework.mock.web.MockHttpServletRequest;
import org.springframework.mock.web.MockServletContext;
 
public class SpringMockTest {
 
@Test
public void testHttpServletRequest(){
	String realPath ="file:D:\\Java\\apache-tomcat-7.0.23\\webapps\\tmn";
	//模拟ServletContext,同时初始化realPath，注意要有file:前缀否则会报错
	MockServletContext context = new MockServletContext(realPath);
	//获取realPath
	System.out.println(context.getRealPath(File.separator));
	//模拟HttpServletRequest
	MockHttpServletRequest request = new MockHttpServletRequest(context);
	//通过HttpServletRequest来获取realPath
	System.out.println(request.getSession().getServletContext().getRealPath(File.separator));
	}
}
```
注意:请在上下文路径的字符串前面加上 **file:** 前缀，否则程序会报错。如上面的程序，realPath的值应为 *file:D:\\Java\\apache-tomcat-7.0.23\\webapps\\tmn* ，若去掉 *file:* 前缀，改为 *D:\\Java\\apache-tomcat-7.0.23\\webapps\\tmn* ，则程序会报错。