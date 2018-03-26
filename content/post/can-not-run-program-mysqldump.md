+++

author = "飞狐"
categories = ["Java编程","数据库"]
tags = ["MySQL","Java"]
date = "2016-03-03T14:33:43+08:00"
description = "Blog of Rosen Lu"
keywords = ["MySQL","Java"]
title = "Cannot run program \"mysqldump\": CreateProcess error=2, The system cannot find the file specified"

+++


项目中用到了`MySQL`数据库的备份功能，通过调用`Java`程序中的**[Runtime](http://docs.oracle.com/javase/6/docs/api/)**来执行`mysqldump`命令自动的生成相关的`MySQL`数据库文件以供恢复之用。相关的代码如下:

```java
Runtime runtime = Runtime.getRuntime();
String mysqlCmd = "mysqldump" + " -u" + username + " -p" + password + 
           "  -h " + databaseAddress + " " +databaseName;
Process process = runtime.exec(mysqlCmd);
```

但是在客户那里实际使用时，有时候会出现在cmd中`MySQL`命令可以正常识别但是程序不能正常执行的情况，报错信息如下:

[//]:(设置前面的内容为summary)
<!--more-->

```java
java.io.IOException: Cannot run program "mysqldump": CreateProcess error=2, The system cannot find the file specified
	at java.lang.ProcessBuilder.start(ProcessBuilder.java:460)
	at java.lang.Runtime.exec(Runtime.java:593)
	at java.lang.Runtime.exec(Runtime.java:431)
	at java.lang.Runtime.exec(Runtime.java:328)
```

Google之后，在**[Stackoverflow](http://stackoverflow.com/)**发现两个相关的问题：

* [Error when backing up MYSQL database](http://stackoverflow.com/questions/15850548/error-when-backing-up-mysql-database)
* [backup mysql database java code](http://stackoverflow.com/questions/13376132/backup-mysql-database-java-code)  

阅读之后，发现上面说问题产生的原因是`mysqldump`命令无法识别，把`mysqldump`可执行文件的路径加入PATH环境变量中即可解决问题。但当我在cmd中无论执行`mysql`或`mysqldump`命令时，都显示这两个命令可以正常执行：  
![mysql1.png](https://ooo.0o0.ooo/2016/03/03/56d7e3254c0a0.png)    
在cmd中输出PATH环境变量时，也显示`MySQL`的bin目录已经添加:  
![mysql2.png](https://ooo.0o0.ooo/2016/03/03/56d7e39ff3ab8.png)    
即使重启电脑，上述通过`Java`备份`MySQL`的代码还是不能正常执行，但当在cmd中执行`mysql`、`mysqldump`命令或输出PATH环境变量时，结果任何上面图片中显示的一致。

这下让我感到很困惑:&nbsp;**通过**`Java`**代码来执行**`mysqldump`**导出操作时去不能正常执行原因是**`MySQL`**的执行路径没有加到PATH环境变量中,但实际检查发现**`MySQL`**的环境变量设置正常，在命令行通过**`mysqldump`**导出sql文件可以成功操作!**继续在网上搜索该问题的解决方案，得到的答案也都是`MySQL`的执行路径没有加到PATH环境变量中去，问题依旧。。。

正当我在为这个问题发愁时，测试部门有个同事的新`Win7`电脑上利用我们的软件执行`MySQL`备份时也出现了类似的问题，之前我还猜有可能是由于客户服务器的操作系统版本太低或某些DLL文件不存在导致的。但现在居然在刚装好的`Win7`电脑上也出现此问题，基本可以排除操作系统的问题。由于在我自己的笔记本和台式研发机上都没出现这个问题，无奈之下我只好把同事的电脑拿过来和我自己的电脑进行对比，看看哪里设置不一样。通过`Win7`中`高级系统设置`查看PATH环境变量，很快就发现了问题的根源：`MySQL`**的执行路径被设置到了**`用户变量`**中的PATH变量里，**`系统变量`**中的PATH变量里却没有**`MySQL`**的执行路径，而**`Java`**代码是匿名执行的，无法获取到**`用户变量`，**只能去**`系统变量`**中寻找相关的可执行命令,因而程序会出错！**  
![mysql3.png](https://ooo.0o0.ooo/2016/03/03/56d7ebcbce7fc.png)  
这下问题原因变得很清楚了，我们在`cmd`中执行`mysql`和`mysqldump`命令以及输出PATH环境变量时，系统会把当前用户的`用户变量`中的PATH和操作系统的`系统变量`中的PATH变量整合到一块，所以我们在cmd中操作时一切正常。但是当我们在`Java`程序中执行`mysqldump`命令时，由于`Java`程序的运行和用户无关，无法获取到`用户变量`中的PATH值，所以当我们在`Java`程序中执行`mysqldump`命令时会出错。这也正好和**[Stackoverflow](http://stackoverflow.com/)**中说明的原因一致。

由于有的电脑上会出现此问题，有的电脑上没有此问题，进一步的深究问题的根源，发现发生问题的电脑和服务器在安装`MySQL`数据库时都是通过我们自己写的`bat`脚本来安装的。而`bat`脚本中设置环境变量的代码如下:

```bash
@echo %Path%
setx PATH "%Path%;C:\INTA\Database\bin;"
@echo %Path%
```

问题的关键就在于***setx PATH "%Path%;C:\INTA\Database\bin;"***这行代码，这样写的话只会把`MySQL`的执行路径加入到当前执行该脚本的`用户变量`中，不会加入到`环境变量`中。而那些没有出问题的电脑都是我自己手动在`系统变量`中设置`MySQL`执行路径的！该问题的解决方法也很简单，在***setx***后面加上***-m***即可，这样`bat`脚本执行时会把`MySQL`的执行路径写入`系统变量`的PATH变量中，不会写入`用户变量`的PATH变量中：

```bash
@echo %Path%
setx -m PATH "%Path%;C:\INTA\Database\bin;"
@echo %Path%
```


Orz~  
想不到由于一个***-m***而让自己郁闷了这么久!