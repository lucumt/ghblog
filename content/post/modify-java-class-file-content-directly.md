+++
date = "2017-08-12T18:09:53+08:00"
title = "在不重新编译的情况下直接修改Java Class文件中的内容"
categories = ["Java编程"]
tags = ["Java","JVM"]
keywords = ["Java"]
author = "飞狐"
description = "在不重新编译的情况下直接修改Java Class文件中的内容"

+++

Java程序实际上执行的是Java文件编译后的Class文件，这是任何一个Java开发人员都了解的基本知识。若Java程序执行的结果不符合要求，通常的解决方法是先修改Java文件，重新编译成Class文件后再次执行。但有时候我们不能直接修改Java文件（如只有包含class文件的jar包），此时我们就只能直接修改Class文件，本文将展示在基于不同的需求通过可视化工具和Javassist库来直接对Class文件进行修改的方法。

<!--more-->
**注：由于直接修改class文件会涉及到class文件结构的相关知识，所以利用此种方式时最好对class文件结构有一定的了解**
## 修改Class文件中的变量
下面的代码为一个典型的输出Hello World的Java小程序
```java
package com.lucumt;

public class Test {
	public static String language = "Java";
	public static void main(String[] args) {
		sayHello();
	}

	public static void sayHello() {
		System.out.println("=====Hello "+language+" World!======");
	}
}
```
在cmd命令行中运行该程序的结果如下   
![未修改之前的运行结果](/blog_img/modify-java-class-file-content-directly/simple-class-before-modifing-running-result.png "未修改之前的Java代码运行结果")  
若想将运行结果从 *Hello Java World* 修改为 *Hello Golang China* ，除了通过修改源代码重新编译运行这个方法之外我们还可以利用工具直接修改原有的class文件来实现。

首先从 **[JBE](http://set.ee/jbe/ "点击链接去下载jbe")** 下载 JBE(Java Bytecode Editor),JBE是一个用于浏览和修改Java Class文件的开源软件，在其官网上可以看到如下图所示的说明信息  
![JBE介绍](/blog_img/modify-java-class-file-content-directly/jbe-official-introduction.png "JBE官方介绍")

下载完该软件后，在该软件中打开我们要修改的Class文件  
![在JBE中打开文件](/blog_img/modify-java-class-file-content-directly/open-class-file-using-jbe.png "在JBE中打开文件")  
首先我们需要将静态变量 *language* 的值从 *Java* 修改为 *Golang*, 由于 *language* 是一个静态变量，故我们需要在class文件的 *clinit* 方法中找到该变量并修改其值。如下图所示，展开 *clinit* 并切换到Code Editor页，可以看到 *language* 的值为 *Java* ，在Code Editor部分将 *Java* 修改为 *Golang* 然后点击Save method即可完成静态变量值的修改。  
![在JBE中修改静态变量值](/blog_img/modify-java-class-file-content-directly/jbe-modify-static-field.png "在JBE中修改静态变量值")  
接着展开 *sayHello* 方法，同样切换到Code Editor页，将 *World* 修改为 *China* 后点击Save method，至此整个修改操作完成。  
![在JBE中修改方法输出值](/blog_img/modify-java-class-file-content-directly/jbe-modify-method-value.png "在JBE中修改方法输出值")  

在命令行中重新执行该程序，输出结果为 *Hello Golang China* ，符合我们的要求。  
![修改之后的运行结果](/blog_img/modify-java-class-file-content-directly/simple-classa-after-modifing-running-result.png "修改之后的Java代码运行结果")    

## 修改Class文件中的方法
对于较为简单的修改需求我们可以利用JBE等工具来直接修改，若要对class文件进行较为复杂的修改，如增加新方法，修改已有方法的实现逻辑等，对于此种需求虽然也可以用JBE实现目的，但工作量很大，容易出错，此时JBE已经不太适合使用，需要寻找其它更快捷的方法。

由于Java文件后生成的class文件是一个包含Java字节码的二进制文件，程序最终执行的就是二进制文件中的字节码，我们的需求可以归纳为如何修改Java字节码文件。前一部分通过JBE来修改class文件只不过是将这个过程进行了图形化封装，我们需要找到更底层的实现方法来适应我们的需求。

此时 **[Javassist](http://jboss-javassist.github.io/javassist/ Javassist官网)** 闪亮登场！在Javassit官网关于其的第一句介绍为 *Javassist (Java Programming Assistant) makes Java bytecode manipulation simple. It is a class library for editing bytecodes in Java* 。Javassist天生就是为修改Java字节码而来的，它提供了源代码和字节码两种级别的API接口，为了实现的简便性，本文主要介绍利用源代码API来修改class文件。

下面的代码为一个计算两个整数相加的程序
```java
package com.lucumt;

public class Test1 {
	public static void main(String[] args) {
          Test1 t1 = new Test1();
          int result = t1.addNumber(3, 5);
          System.out.println("result is: "+result);
	}
	
	public int addNumber(int a,int b){
		return a+b;
	}
}
```
正常情况下，其输出结果如下  
![未修改方法前的运行结果](/blog_img/modify-java-class-file-content-directly/java-method-before-modifing-running-result.png "未修改方法前的运行结果")  
若我们想将 *addNumber* 的返回结果从两个数之和变为两个数立方后求和，则可以利用Javassist提供的API通过Java程序来直接修改class文件。

关于如何使用Javassist，请直接参看相应的 **[入门教程](http://jboss-javassist.github.io/javassist/tutorial/tutorial.html)** ，本文不再详细说明，利用Javassist修改 *addNumber* 的Java代码如下：
```java
package com.lucumt.test;

import java.io.IOException;

import javassist.CannotCompileException;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
import javassist.NotFoundException;

public class UpdateMethod {

	public static void main(String[] args) {
		updateMethod();
	}
	
	public static void updateMethod(){
		try {
			ClassPool cPool = new ClassPool(true);
		        //如果该文件引入了其它类，需要利用类似如下方式声明
			//cPool.importPackage("java.util.List");
			
			//设置class文件的位置
			cPool.insertClassPath("D:\\Java\\eclipse\\newworkspace\\test\\bin");
			
			//获取该class对象
			CtClass cClass = cPool.get("com.lucumt.Test1");
			
			//获取到对应的方法
			CtMethod cMethod = cClass.getDeclaredMethod("addNumber");
			
			//更改该方法的内部实现
			//需要注意的是对于参数的引用要以$开始，不能直接输入参数名称
			cMethod.setBody("{ return $1*$1*$1+$2*$2*$2; }");
			
			//替换原有的文件
			cClass.writeFile("D:\\Java\\eclipse\\newworkspace\\test\\bin");
			
			System.out.println("=======修改方法完=========");
		} catch (NotFoundException e) {
			e.printStackTrace();
		} catch (CannotCompileException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}

	}
}
```
运行该代码后重新执行 *Test1* 后的结果如下，从图中可以看出运行结果符合预期   
![修改方法后的运行结果](/blog_img/modify-java-class-file-content-directly/java-method-after-modifing-runnning-result.png "修改方法后的运行结果")  

关于 *UpdateMethod* 工具类有如下几点说明：

* 如果要修改的class文件中引入了其它类，需要调用 *ClassPool* 中的 *importPackage* 方法引入该类，否则程序会报错
* 修改完后，一定要调用 *CtClass* 中的 *writeFile* 方法覆盖原有的class文件，否则修改不生效
* 在修改方法的过程中若要引用方法参数，不能在修改程序代码中直接写该参数，否则程序会抛出*javassist.CannotCompileException: [source error] no such field:* 异常。在本例中 *addNumber* 的两个参数分别为 *a* 和 *b* ，在修改时不能写成`cMethod.setBody("{ return a*a*a+b*b*b; }")`需要修改为`cMethod.setBody("{ return $1*$1*$1+$2*$2*$2; }")`
* 在Javassist的 **[Introspection and customization](http://jboss-javassist.github.io/javassist/tutorial/tutorial2.html#intro)** 部分有如下一段话    
*The parameters passed to the target method are accessible with $1, $2, ... instead of the original parameter names. $1 represents the first parameter, $2 represents the second parameter, and so on. The types of those variables are identical to the parameter types. $0 is equivalent to this. If the method is static, $0 is not available.*  
从中可知，方法中的参数从 *$1* 开始，若该方法为非 *static* 方法，可以用 *$0* 来表示该方法实例自身，若该方法为 *static* 方法，则 *$0* 不可用

## 在Class文件中增加方法

Javassist不仅可以修改已有的方法，还可以给class文件增加新的方法。仍以前面的 *Test1* Java代码中为例，现要求增加一个名为 *showParameter* 的方法并在 *addNumber* 方法中调用，其主要功能是输出 *addNumber* 中传入的参数。利用Javassist修改class文件实现该功能的代码如下  
```java
package com.lucumt.test;

import java.io.IOException;

import javassist.CannotCompileException;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
import javassist.CtNewMethod;
import javassist.NotFoundException;

public class AddMethod {

	public static void main(String[] args) {
		addMethod();
	}
	
	public static void addMethod(){
		try {
			ClassPool cPool = new ClassPool(true);
			cPool.insertClassPath("D:\\Java\\eclipse\\newworkspace\\test\\bin");
			CtClass cClass = cPool.get("com.lucumt.Test1");
			
			
			CtMethod cMethod = cClass.getDeclaredMethod("addNumber");
			
			//增加一个新方法
			String methodStr ="public void showParameters(int a,int b){" 
					    +"  System.out.println(\"First parameter: \"+a);"
					    +"  System.out.println(\"Second parameter: \"+b);"
					    +"}";
			CtMethod newMethod = CtNewMethod.make(methodStr, cClass);
			cClass.addMethod(newMethod);
			
			//调用新增的方法
			cMethod.setBody("{ showParameters($1,$2);return $1*$1*$1+$2*$2*$2; }");
			cClass.writeFile("D:\\Java\\eclipse\\newworkspace\\test\\bin");
			
		} catch (NotFoundException e) {
			e.printStackTrace();
		} catch (CannotCompileException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}

	}
}
```  
运行该代码后重新执行 *Test1* 后的结果如下，从图中可以看出运行结果符合预期     
![新增方法后的运行结果](/blog_img/modify-java-class-file-content-directly/java-add-method-runnning-result.png "新增方法后的运行结果")  
从上述代码可以看出，利用Javassist增加方法比修改方法更简单，先将要新增的方法内容赋值到字符串，然后分别调用相关类的 *make* 和 *addMethod* 方法即可。  

## 后记
利用JBE或Javassist虽然可以实现直接修改class文件的内容，但毕竟属于不正规的做法，可能会导致后续版本不一致等问题，在条件允许的情况下还是要尽量通过修改Java文件然后重新编译的方式来实现目的。

