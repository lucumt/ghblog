---
title: "在模板类中利用抽象类或接口default方法实现默认方法以简化开发与维护"
date: 2018-06-02T13:27:06+08:00
lastmod: 2018-06-02T13:27:06+08:00
draft: false
keywords: ["java","抽象类","接口","设计模式","默认方法"]
description: "简要说明如何在Java中利用抽象类或接口default方法实现默认方法来简化模板设计模式，避免代码冗余，提高可读性与可维护性"
tags: ["java"]
categories: ["java编程"]
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

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

在开发大规模的`Java`项目时，通常会涉及到各种设计模式，本文简要说明当使用[**模板方法**](https://zh.wikipedia.org/zh-cn/%E6%A8%A1%E6%9D%BF%E6%96%B9%E6%B3%95)时，如何利用接口default方法或抽象类对具体的子类进行精简，便于开发与维护。

<!--more-->

## 背景说明

假设部门要进行一次团建，主要包含如下3个活动：

* 轰趴(play)
* 聚餐(eat)
* 唱歌(sing)

这3个按照时间顺序先后进行，大家可以选择参加启动的一个或多个，则可以创建一个类似如下的类来表示上述流程

```java
public abstract class TeamBuilding {

    public void participate() {
        play();
        eat();
        sing();
    }

    public abstract void play();

    public abstract void eat();

    public abstract void sing();

}
```

上述代码中`participate()`方法表示参加此次活动，里面包含了3个抽象方法，表示 邀请了所有人，被邀请人员可以选择性参加其中一个或多个活动。

按照正常思维实际使用时需要继承该类并分别实现该类中的方法，即使不参加其中某一项活动，对应的实现类参考如下

```java
public class PersonA extends TeamBuilding {

    @Override
    public void play() {
        System.out.println("play card");
    }

    @Override
    public void eat() {
        System.out.println("eat cake");
    }

    @Override
    public void sing() {
        // 不参加该项活动，实现为空方法
    }
}

public class PersonB extends TeamBuilding {

    @Override
    public void play() {
        // 不参加该项活动，实现为空方法
    }

    @Override
    public void eat() {
        System.out.println("eat cake");
    }

    @Override
    public void sing() {
        System.out.println("sing a song");
    }
}

public class PersonC extends TeamBuilding {

    @Override
    public void play() {
        System.out.println("play basketball");
    }

    @Override
    public void eat() {
        // 不参加该项活动，实现为空方法
    }

    @Override
    public void sing() {
        System.out.println("sing a song");
    }
}
```

可以看出相关的实现类中，有很多空的实现方法，形成了冗余，不便于维护。

同时在模板方法中每增加一个步骤，所有的子类中都需要实现方法，改动量大，也不便于维护。

```java
public abstract class TeamBuilding {

    public void participate() {
        play();
        drink();
        eat();
        sing();
    }

    public abstract void play();

    // 新增一个方法后，对应的子类都需要实现该方法
    public abstract void drink();

    public abstract void eat();

    public abstract void sing();

}
```

期望实现的效果是对应的子类中是实现自己关注的流程，且添加新的步骤时，子类不用全部改动，类似如下

```java
public class PersonA extends TeamBuilding {

    @Override
    public void play() {
        System.out.println("play card");
    }

    @Override
    public void eat() {
        System.out.println("eat cake");
    }
    
}

public class PersonB extends TeamBuilding {

    @Override
    public void eat() {
        System.out.println("eat cake");
    }

    @Override
    public void sing() {
        System.out.println("sing a song");
    }
}

public class PersonC extends TeamBuilding {

    @Override
    public void play() {
        System.out.println("play basketball");
    }

    @Override
    public void sing() {
        System.out.println("sing a song");
    }
}
```

接下来以抽象类和接口默认方法分别说明如何设计父类以便达成上述目的。

## 抽象类实现

仿照`Java`多线程编程中的[**AbstractQueuedSynchronizer**](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/AbstractQueuedSynchronizer.html)类里面的获取锁与释放锁的实现，在父类中给所有的方法都取消`abstract`修饰符，默认用[**UnsupportedOperationException**](https://docs.oracle.com/javase/8/docs/api/java/lang/UnsupportedOperationException.html)实现全部方法，子类按需`override`即可。

`AbstractQueuedSynchronizer`相关实现 ：

```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}

protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```

相关代码如下：

```java
public abstract class TeamBuilding {

    public void participate() {
        play();
        eat();
        sing();
    }

    protected void play() {
        throw new UnsupportedOperationException();
    }

    protected void eat() {
        throw new UnsupportedOperationException();
    }

    protected void sing() {
        throw new UnsupportedOperationException();
    }

}
```

## 接口实现

在`JDK8`和以上版本中，也可将抽象类修改为接口，利用[**接口默认方法**](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)实现，相关代码如下：

```java
public interface TeamBuilding {

    default void participate() {
        play();
        eat();
        sing();
    }

    default void play() {
        throw new UnsupportedOperationException();
    }

    default void eat() {
        throw new UnsupportedOperationException();
    }

    default void sing() {
        throw new UnsupportedOperationException();
    }

}
```

对应的子类实现如下：

```java
public class PersonA implements TeamBuilding {

    @Override
    public void play() {
        System.out.println("play card");
    }

    @Override
    public void eat() {
        System.out.println("eat cake");
    }

}

public class PersonB implements TeamBuilding {

    @Override
    public void eat() {
        System.out.println("eat cake");
    }

    @Override
    public void sing() {
        System.out.println("sing a song");
    }
}

public class PersonC implements TeamBuilding {

    @Override
    public void play() {
        System.out.println("play basketball");
    }

    @Override
    public void sing() {
        System.out.println("sing a song");
    }
}
```

