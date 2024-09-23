---
title: "Spring中BeanFactory和FactoryBean的区别以及使用场景"
date: 2022-01-16T11:00:01+08:00
lastmod: 2022-01-16T11:00:01+08:00
draft: false
keywords: ["Spring","BeanFactory","FactoryBean","区别","使用场景"]
description: "简要叙述Spring中BeanFactory和FactoryBean的区别以及使用场景"
tags: ["Spring"]
categories: ["Spring系列"]
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

[**BeanFactory**](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html)和[**FactoryBean**](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/FactoryBean.html)的区别与使用场景是[**Spring**](https://spring.io/)面试中的高频题之一，本文基于网上资料和个人理解，简要说明他们之间的异同以及使用场景。

<!--more-->

## 组件说明

### BeanFactory

在`BeanFactory`的API中，一开始就有如下一句描述

> The root interface for accessing a Spring bean container

这句话指明了`BeanFactory`在`Spring`家族中的地位，它是**操作`Spring`容器、获取`Spring` Bean的根接口**。从字面意思可以看出它是一个Bean工厂，其内部采用了工厂模式，通过它们我们可进行创建Bean、获取Bean等操作[^1]。

在学习与使用`Spring`过程中接触到的`ApplicaitonContext`、`AnnotationConfigApplicastionContext`、`ClassPathXmlApplicationContext`等都是其子接口和具体的实现类，它是我们在`Spring`框架中接触最多的系统接口，下图中的UML展示了`BeanFactory`及其子接口和实现类之间的关系[^2]

![BeanFactory UML类图](/blog_img/spring/difference-between-factorybean-and-beanfactory/bean-factory-uml-1.png "BeanFactory UML类图")  

下述代码展示了`BeanFactory`所包含的全部15个接口方法，它们涵盖了`Spring`中对Bean对象的各种读取操作，若经常使用`Spring`这些接口方法看起来就会很熟悉。

<details>
  <summary><mark><font color=darkred>BeanFactory接口(点击可展开)</font></mark></summary>


  ```java
  public interface BeanFactory {
  
  	String FACTORY_BEAN_PREFIX = "&";
      
  	Object getBean(String name) throws BeansException;
  
  	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
  
  	Object getBean(String name, Object... args) throws BeansException;
  
  	<T> T getBean(Class<T> requiredType) throws BeansException;
  
  	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
  
  	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
  
  	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
  
  	boolean containsBean(String name);
  
  	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
      
  	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
      
  	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
  
  	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
  
  	@Nullable
  	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
          
  	@Nullable
  	Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;
  
  	String[] getAliases(String name);
  }
  ```

</details>

由于`BeanFactory`是操作`Spring`容器和对象的根接口，`Spring`官方要求它尽可能的支持Bean生命周期的各种接口，基于`Spring`官方API的描述，其完整的方法链和顺序如下：

1. BeanNameAware's `setBeanName`
2. BeanClassLoaderAware's `setBeanClassLoader`
3. BeanFactoryAware's `setBeanFactory`
4. EnvironmentAware's `setEnvironment`
5. EmbeddedValueResolverAware's `setEmbeddedValueResolver`
6. ResourceLoaderAware's `setResourceLoader` (only applicable when running in an application context)
7. ApplicationEventPublisherAware's `setApplicationEventPublisher` (only applicable when running in an application context)
8. MessageSourceAware's `setMessageSource` (only applicable when running in an application context)
9. ApplicationContextAware's `setApplicationContext` (only applicable when running in an application context)
10. ServletContextAware's `setServletContext` (only applicable when running in a web application context)
11. `postProcessBeforeInitialization` methods of BeanPostProcessors
12. InitializingBean's `afterPropertiesSet`
13. a custom `init-method` definition
14. `postProcessAfterInitialization` methods of BeanPostProcessors

上述初始化链可很好的帮我们回答面试中另外几个常见的问题：

* `Spring`中Bean如何初始化
* 描述`Spring`中Bean的生命周期

### FactoryBean

`FactoryBean`从名字就能看出来是一个Bean，但不是普通的Bean，否则`Spring`作者为啥要单独的创建这个接口呢？`FactoryBean`也是一个接口，其源码如下，从中可以看出该接口只有3个方法，与`BeanFactory`相比数量大为减少，同时这3个接口方法的作用也同`BeanFactory`中的相关方法类似。

```java
public interface FactoryBean<T> {

	String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

	@Nullable
	T getObject() throws Exception;

	@Nullable
	Class<?> getObjectType();

	default boolean isSingleton() {
		return true;
	}

}
```

通过上述代码能够获取的信息依旧有限，在`FactoryBean`的官方API中有如下描述 

>Interface to be implemented by objects used within a [`BeanFactory`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html) which are themselves factories for individual objects. If a bean implements this interface, it is used as a factory for an object to expose, not directly as a bean instance that will be exposed itself.
>
>**NB: A bean that implements this interface cannot be used as a normal bean.** A FactoryBean is defined in a bean style, but the object exposed for bean references ([`getObject()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/FactoryBean.html#getObject--)) is always the object that it creates.

上述文档的主要结论如下：

* `FactoryBean`不是一个普通的Bean，它实际上是一个用于创建特定bean的工厂
* 通过实现`FactoryBean`接口，可通过暴露对象的方式创建特定bean对象，而这个bean对象本身不会暴露

到这里虽然二者的区别清楚了，但是`FactoryBean`的使用场景还是不清晰，在[**https://spring.io/blog/2011/08/09/what-s-a-factorybean**](https://spring.io/blog/2011/08/09/what-s-a-factorybean)这篇文章中，`Spring`的作者之一[**JOSH LONG**](https://spring.io/team/joshlong)用如下文字阐述了`FactoryBean`的使用场景

> A `FactoryBean` is a pattern to encapsulate interesting object construction logic in a class. It might be used, for example, to encode the construction of a complex object graph in a reusable way. Often this is used to construct complex objects that have many dependencies. It might also be used when the construction logic itself is highly volatile and depends on the configuration. A `FactoryBean` is also useful to help Spring construct objects that it couldn’t easily construct itself.

从上述文字中可以看出`FactoryBean`主要**用于构建一些实例化过程较为复杂或有配置依赖的对象(即非普通POJO)，并将其交给`Spring`容器管理**。

`Spring`框架本身就自带了实现`FactoryBean`的70多个接口，如`ProxyFactoryBean`、`MapFactoryBean`、`PropertiesFactoryBean`等，从个人角度来看它们要么理解起来较为复杂，要么使用较少不具有代表性，下面以`MyBatis-Spring`中的[**SqlSessionFactoryBean**](https://github.com/mybatis/spring/blob/master/src/main/java/org/mybatis/spring/SqlSessionFactoryBean.java)为例来了解其用法

```java
  @Override
  public SqlSessionFactory getObject() throws Exception {
    if (this.sqlSessionFactory == null) {
      afterPropertiesSet();
    }

    return this.sqlSessionFactory;
  }

  // 参数校验与对象创建
  public void afterPropertiesSet() throws Exception {
    notNull(dataSource, "Property 'dataSource' is required");
    notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
    state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),
        "Property 'configuration' and 'configLocation' can not specified with together");

    this.sqlSessionFactory = buildSqlSessionFactory();
  }

// 基于配置文件创建具体的SqlSessionFactory
protected SqlSessionFactory buildSqlSessionFactory() throws Exception {
    // 各种检查代码
    
    return this.sqlSessionFactoryBuilder.build(targetConfiguration);
}
```

在创建`SqlSessionFactory`时需要依赖数据库的配置等一些配置信息，但这些配置信息若通过`BeanFactory`以传统的**实例化**->**初始化**的方式创建时是很难办到的，虽然可以通过`BeanPostProcessors`的接口中的回调方法来实现，但会导致代码复杂并且不能满足某些特殊场景下的需求，而通过`FactoryBean`我们只需要根据实际业务逻辑在`getObject()`方法中创建对应的对象并返回即可，由于对象的创建由我们自己把控，在达到代码简洁的同时，创建的对象也能被`Spring`容器管理。

### 使用示例

假设有如下所示的Group类

```java
@Service
public class Group {

    private String groupName;

    // getter setter
}
```

在使用`BeanFactory`获取对象的方法如下

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
    Group group = context.getBean(Group.class);
    System.out.println(group.hashCode());
}
```

输出结果为Group@25d250c6，可以看到已经正确的获取到了Bean对象。

定义一个实现了`FactoryBean`接口的GroupFactoryBean，其代码如下：

```java
public class GroupFactoryBean implements FactoryBean<Group> {


    @Override
    public Group getObject() throws Exception {
        return new Group();
    }

    @Override
    public Class<?> getObjectType() {
        return Group.class;
    }
}
```

测试代码如下：

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(GroupFactoryBean.class);
    Object object = context.getBean("groupFactoryBean");
    System.out.println(object);
}
```

输出结果为Group@fdefd3f，如前面通过`BeanFactory` 创建的对象类似，接下来将**groupFactoryBean**添加一个`&`前缀，变为`&groupFactoryBean`后再次进行测试

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(GroupFactoryBean.class);
    Object object = context.getBean("&groupFactoryBean");
    System.out.println(object);
}
```

输出结果为GroupFactoryBean@1990a65e，由此可得出如下结论 ：

* 当直接通过`FactoryBean`实现类的接口名称获取对象时，得到的是该`FactoryBean`实现类中创建的对象本身，也是我们正常的使用方式
* 当在`FactoryBean`接口对象名前面加上`&`前缀时，得到的是`FactoryBean`实现类本身，一般用于调试分析。

## 二者之间的区别

* 相同点：
  * 都是接口类，需要使用者自己实现相应的方法
  * 接口中创建bean对象都能被`Spring`容器管理
* 不同点:
  * `BeanFactory`提供的接口方法更多，更具有灵活性，如能获取别名、bean对象类型等
  * `BeanFactory`基于`Spring`规范实现，其创建的bean会在`Spring`容器中经历完整的生命周期，能够调用`@PostConstruct`、`BeanPostProcessors`等接口和方法
  * `Spring`容器只负责`FactoryBean`创建的bean对象的生命周期管理，不负责bean对象的创建和销毁(因为由我们自己实现了嘛!)，所以调用`@PostConstruct`和`@PreDestroy`方法不会生效，需要实现`DisposableBean`等接口来达到该目的

## 相关使用场景

* **`BeanFactory`**:
  * 项目中各种常规的`POJO`、`Dao`、`Service`等不涉及复杂构造的场景
* **`FactoryBean`**:
  * 对象的构造(实例化)较为复杂，通常是系统中的一些底层组件，如`SqlSessionFactoryBean`、`ProxyFactoryBean`等

## 参考文档

* https://www.cnblogs.com/aspirant/p/9082858.html
* https://spring.io/blog/2011/08/09/what-s-a-factorybean

* https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/FactoryBean.html



[^1]: 实际上Bean的创建、销毁都是由容器来负责的，使用时更多关注的是Bean对象的配置以及如何获取Bean对象
[^2]: 出于篇幅的考虑，实际上这张UML类图展示的并不完整，可在IDEA中基于`BeanFactory`自己生成完整的UML图
