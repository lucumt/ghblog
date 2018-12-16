---
title: "Java中利用==和equals()进行字符串比较的说明"
date: 2018-12-08T23:02:10+08:00
lastmod: 2018-12-08T23:02:10+08:00
draft: true
keywords: ["Java","==","equals"]
description: "Java中字符串的比较说明"
tags: ["Java"]
categories: ["Java编程"]
author: "Rosen Lu"

---

Java中字符串的比较在面试中很常见，我们都知道比较字符串是否相等要使用`equals()`而不是`==`。本文首先利用`javap`命令从class文件的角度来分析不同字符串比较的结果，然后分析下`Tomcat`中如何获取前端输入的字符串参数,并以此说明Java Web开发中该如何正确的进行字符串的比较。 

<!--more-->

## 简单字符串比较

测试代码如下：
```java
public class StringTest {

	public static void main(String[] args) {
		String s1 = "Hello World";
		String s2 = "Hello World";
		String s3 = new String("Hello World");
		String s4 = new String("Hello World");
		
		System.out.println("利用==比较");
		System.out.println(s1 == s2);
		System.out.println(s1 == s4);
		System.out.println(s3 == s4);
		System.out.println(s1 == s4.intern());
		System.out.println(s3.intern() == s4.intern());
		
		System.out.println("\n利用equals()比较");
		System.out.println(s1.equals(s2));
		System.out.println(s1.equals(s4));
		System.out.println(s3.equals(s4));
	}
}
```
程序运行的结果如下：  
![简单字符串比较](/blog_img/java-string-equal-compare/string_compare_result_1.png "简单字符串比较")  
从上图中可以看出: 利用`equals()`比较时返回的结果全为true,而利用`==`比较的结果只有部分为true。利用`javap`命令输出class文件内容如下(省略掉了`System.out.println()相关的`)

```java
Compiled from "StringTest.java"
public class StringTest {
  public StringTest();
    Code:
       0: aload_0
       1: invokespecial #8                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #16                 // String Hello World
       2: astore_1
       3: ldc           #16                 // String Hello World
       5: astore_2
       6: new           #18                 // class java/lang/String
       9: dup
      10: ldc           #16                 // String Hello World
      12: invokespecial #20                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
      15: astore_3
      16: new           #18                 // class java/lang/String
      19: dup
      20: ldc           #16                 // String Hello World
      22: invokespecial #20                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
      25: astore        4
      27: new           #18                 // class java/lang/String
      30: dup
      31: ldc           #16                 // String Hello World
      33: invokespecial #20                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
      36: invokevirtual #23                 // Method java/lang/String.intern:()Ljava/lang/String;
      39: astore        5
      //......
     214: return
}
```
为了能读懂其内容，可先从[The Java® Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se8/html)中了解相关的指令，本文将涉及到的指令列举如下

* **ldc**，将字符串从运行时常量池压入操作栈中
* **astore**，将一个数值从操作栈存入局部变量表
* **dup**，复制栈顶的数值并将复制的数值重新压入栈中
* **invokespecial**，调用实例构造器<init>方法、私有方法和父类方法
* **invokevirtual**，调用实例方法，基于类进行分发

基于上述命令我们可以发现字符串`s1`、`s2`、`s5`都是从常量池中的获取的，而`s3`、`s4`则是分别创建了两个`String`对象，如下图所示。

在Java中`==`比较的是内存地址是否相同，而`equals()`比较的是其文本值是否相同，而从常量池中多次获取同一个常量其地址是相同的，新建的`String`对象JVM会为其重新分配内存地址。故在利用`==`进行比较时,1、2、5这三个都是基于常量池的比较，它们的结果都为true，而3、4种都包含有`String`对象，故其结果均为false。

![class文件分析1](/blog_img/java-string-equal-compare/string_class_content_analysis_1.png "单纯的字符串比较时class文件分析")  

## 字符串相加后比较

将上述代码修改为如下:
```java
public class StringTest {

	public static void main(String[] args) {
		String s1 = "Hello World";
		String s2 = "Hello ";
		String s3 = s2 + "World";
		String s4 = "Hello " + "World";
		String s5 = "Hello " + new String("World");
		String s6 = "Hello " + new String("World").intern();
		
		System.out.println("利用==比较:");
		System.out.println(s1 == s3);
		System.out.println(s1 == s4);
		System.out.println(s1 == s5);
		System.out.println(s1 == s6);
		
		System.out.println("\n利用equals()比较:");
		System.out.println(s1.equals(s3));
		System.out.println(s1.equals(s4));
		System.out.println(s1.equals(s5));
		System.out.println(s1.equals(s6));
	}
}
```
程序运行的结果如下:  
![字符串相加后的比较](/blog_img/java-string-equal-compare/string_compare_result_2.png "字符串相加后的比较结果")  

此时利用`==`比较的结果只有1个为true，为了探究原因需要继续分析class文件内容，用`javap`命令输出的class文件内容如下(省略掉了`System.out.println()相关的`)

```java
Compiled from "StringTest.java"
public class StringTest {
  public StringTest();
    Code:
       0: aload_0
       1: invokespecial #8                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #16                 // String Hello World
       2: astore_1
       3: ldc           #18                 // String Hello
       5: astore_2
       6: new           #20                 // class java/lang/StringBuilder
       9: dup
      10: aload_2
      11: invokestatic  #22                 // Method java/lang/String.valueOf:(Ljava/lang/Object;)Ljava/lang/String;
      14: invokespecial #28                 // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
      17: ldc           #31                 // String World
      19: invokevirtual #33                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      22: invokevirtual #37                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      25: astore_3
      26: ldc           #16                 // String Hello World
      28: astore        4
      30: new           #20                 // class java/lang/StringBuilder
      33: dup
      34: ldc           #18                 // String Hello
      36: invokespecial #28                 // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
      39: new           #23                 // class java/lang/String
      42: dup
      43: ldc           #31                 // String World
      45: invokespecial #41                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
      48: invokevirtual #33                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      51: invokevirtual #37                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      54: astore        5
      56: new           #20                 // class java/lang/StringBuilder
      59: dup
      60: ldc           #18                 // String Hello
      62: invokespecial #28                 // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
      65: new           #23                 // class java/lang/String
      68: dup
      69: ldc           #31                 // String World
      71: invokespecial #41                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
      74: invokevirtual #42                 // Method java/lang/String.intern:()Ljava/lang/String;
      77: invokevirtual #33                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      80: invokevirtual #37                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      83: astore        6
      //....
     215: return
}
```
基于class文件的内容对`s3`、`s4`、`s5`、`s6`这4个字符串进行分析，可发现`s4`是编译器自动优化后从字符常量池中获取的之外，其余的3个字符串都是利用`StringBuilder`中的`toString()`方法生成的，`toString()`的源码如下，可以看出返回的是一个`String`对象。这正好解释了除了`s1==s4`输出值为true之外其余的输出值都为false的原因。

```java
@Override
public String toString() {
    return new String(value, 0, count);
}
```

![class文件分析2](/blog_img/java-string-equal-compare/string_class_content_analysis_2.png "相加的字符串比较时class文件分析")  

## String是不可变的原因分析

在学习Java时我们一直被强调`String`是不可变的，而实际使用中我们又可以利用类似如下的代码对`String`进行拼接操作，看起来很矛盾。
```java
public class StringTest {

	public static void main(String[] args) {
		String s1 = "Hello";
		s1 +=" Java";
		s1 +=" Golang";
	}
}
```
同样可以通过阅读class文件来分析该问题，对应的class文件如下:
```java
Compiled from "StringTest.java"
public class StringTest {
  public StringTest();
    Code:
       0: aload_0
       1: invokespecial #8                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #16                 // String Hello
       2: astore_1
       3: new           #18                 // class java/lang/StringBuilder
       6: dup
       7: aload_1
       8: invokestatic  #20                 // Method java/lang/String.valueOf:(Ljava/lang/Object;)Ljava/lang/String;
      11: invokespecial #26                 // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
      14: ldc           #29                 // String  Java
      16: invokevirtual #31                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      19: invokevirtual #35                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      22: astore_1
      23: new           #18                 // class java/lang/StringBuilder
      26: dup
      27: aload_1
      28: invokestatic  #20                 // Method java/lang/String.valueOf:(Ljava/lang/Object;)Ljava/lang/String;
      31: invokespecial #26                 // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
      34: ldc           #39                 // String  Golang
      36: invokevirtual #31                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      39: invokevirtual #35                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      42: astore_1
      43: return
}
```
分析class文件可发现利用`+=`操作实质上是通过`StringBuilder`来拼接并重新构建字符串，每次`+=`操作都会生成新的字符串，原始字符串的指向地址被丢失，`String`是不可变实际上指的是原有的字符串无法改变，前述的问题得以解决。

同时通过分析上述class文件还能得出以下结论:

* 对于类似`String str="Hello" + "World"`的赋值，JVM会将其自动优化为一个字符串常量，除此之外的其它基于`String`的拼接都会生成新的`String`对象；
* 在字符串拼接时，若采用基于`String`的拼接操作，会频繁的创建`StringBuilder`对象，影响程序性能，应该采用`StringBuilder`替代以减少`StringBuilder`创建的次数;
* 需要确保线程安全时，可以使用`StringBuffer`替代`StringBuilder`

## Java Web程序中的字符串赋值

利用下述代码在Web页面输入用户名和密码，然后利用`==`在Servlet代码中和特定的字符串进行比较。

```html
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>测试数据传递</title>
</head>
<body>
 
  <div>
    <form action="addUser" method="post">
       <table>
         <tbody>
           <tr>
             <td>用户名:</td>
             <td><input type="text" name="username"/></td>
           </tr>
           <tr>
             <td>密码:</td>
             <td><input type="password" name="password"/></td>
           </tr>
           <tr>
             <td>&nbsp;</td>
             <td><button type="submit">提交</button></td>
           </tr>
         </tbody>
       </table>
 
    </form>
  </div>
</body>
</html>
```

```java
TestServlet.java
public class TestServlet extends HttpServlet {
 
	private static final long serialVersionUID = 6174437812832777462L;
	@Override
	protected void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		doPost(request, response);
	}
	 
	@Override
	protected void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		String username = request.getParameter("username");
		System.out.println(username == "Rosen");
		System.out.println("Rosen".equals(username));
		request.getRequestDispatcher("index.html").forward(request, response);
	}
 
}
```

在`Tomcat7`中的运行结果如下，可以看出其运行结果符合前面基于class文件的理论分析。
![Web程序中的字符串比较](/blog_img/java-string-equal-compare/string_compare_result_3.png "Web程序中的字符串比较")  

通过前面的分析可知通过Web服务器传递给Servlet的字符串参数肯定是一个`String`对象而非一个字符串常量。接下来通过在GitHub中分析**[Tomcat](https://github.com/apache/tomcat)**源码来了解其如何赋值。

1. 在`Tomcat`中，生成参数的相关代码位于**[Parameters.java](https://github.com/apache/tomcat/blob/trunk/java/org/apache/tomcat/util/http/Parameters.java)**类中`private void processParameters(byte bytes[], int start, int len, Charset charset)`方法中，该方法给参数赋值的核心代码如下：
```java
if (valueStart >= 0) {
    if (decodeValue) {
        urlDecode(tmpValue);
    }
    tmpValue.setCharset(charset);
    value = tmpValue.toString();
} else {
    value = "";
}
```  

2. 继续查看可知`tmpValue`的类型为**[ByteChunk](https://github.com/apache/tomcat/blob/trunk/java/org/apache/tomcat/util/buf/ByteChunk.java)**,其`toString()`核心代码如下：
```java
public String toString() {
    if (isNull()) {
        return null;
    } else if (end - start == 0) {
        return "";
    }
    return StringCache.toString(this);
}
```

3. 继续查看**[StringCache](https://github.com/apache/tomcat/blob/trunk/java/org/apache/tomcat/util/buf/StringCache.java)**的`toString()`方法如下
```java
public static String toString(ByteChunk bc) {

    // If the cache is null, then either caching is disabled, or we're
    // still training
    if (bcCache == null) {
        String value = bc.toStringInternal();
        if (byteEnabled && (value.length() < maxStringSize)) {
            // If training, everything is synced
            synchronized (bcStats) {
                // If the cache has been generated on a previous invocation
                // while waiting for the lock, just return the toString
                // value we just calculated
                if (bcCache != null) {
                    return value;
                }
                // Two cases: either we just exceeded the train count, in
                // which case the cache must be created, or we just update
                // the count for the string
                if (bcCount > trainThreshold) {
                    long t1 = System.currentTimeMillis();
                    // Sort the entries according to occurrence
                    TreeMap<Integer,ArrayList<ByteEntry>> tempMap =
                            new TreeMap<>();
                    for (Entry<ByteEntry,int[]> item : bcStats.entrySet()) {
                        ByteEntry entry = item.getKey();
                        int[] countA = item.getValue();
                        Integer count = Integer.valueOf(countA[0]);
                        // Add to the list for that count
                        ArrayList<ByteEntry> list = tempMap.get(count);
                        if (list == null) {
                            // Create list
                            list = new ArrayList<>();
                            tempMap.put(count, list);
                        }
                        list.add(entry);
                    }
                    // Allocate array of the right size
                    int size = bcStats.size();
                    if (size > cacheSize) {
                        size = cacheSize;
                    }
                    ByteEntry[] tempbcCache = new ByteEntry[size];
                    // Fill it up using an alphabetical order
                    // and a dumb insert sort
                    ByteChunk tempChunk = new ByteChunk();
                    int n = 0;
                    while (n < size) {
                        Object key = tempMap.lastKey();
                        ArrayList<ByteEntry> list = tempMap.get(key);
                        for (int i = 0; i < list.size() && n < size; i++) {
                            ByteEntry entry = list.get(i);
                            tempChunk.setBytes(entry.name, 0,
                                    entry.name.length);
                            int insertPos = findClosest(tempChunk,
                                    tempbcCache, n);
                            if (insertPos == n) {
                                tempbcCache[n + 1] = entry;
                            } else {
                                System.arraycopy(tempbcCache, insertPos + 1,
                                        tempbcCache, insertPos + 2,
                                        n - insertPos - 1);
                                tempbcCache[insertPos + 1] = entry;
                            }
                            n++;
                        }
                        tempMap.remove(key);
                    }
                    bcCount = 0;
                    bcStats.clear();
                    bcCache = tempbcCache;
                    if (log.isDebugEnabled()) {
                        long t2 = System.currentTimeMillis();
                        log.debug("ByteCache generation time: " +
                                (t2 - t1) + "ms");
                    }
                } else {
                    bcCount++;
                    // Allocate new ByteEntry for the lookup
                    ByteEntry entry = new ByteEntry();
                    entry.value = value;
                    int[] count = bcStats.get(entry);
                    if (count == null) {
                        int end = bc.getEnd();
                        int start = bc.getStart();
                        // Create byte array and copy bytes
                        entry.name = new byte[bc.getLength()];
                        System.arraycopy(bc.getBuffer(), start, entry.name,
                                0, end - start);
                        // Set encoding
                        entry.charset = bc.getCharset();
                        // Initialize occurrence count to one
                        count = new int[1];
                        count[0] = 1;
                        // Set in the stats hash map
                        bcStats.put(entry, count);
                    } else {
                        count[0] = count[0] + 1;
                    }
                }
            }
        }
        return value;
    } else {
        accessCount++;
        // Find the corresponding String
        String result = find(bc);
        if (result == null) {
            return bc.toStringInternal();
        }
        // Note: We don't care about safety for the stats
        hitCount++;
        return result;
    }
}
```
该方法篇幅很长，但核心代码只有一行`String value = bc.toStringInternal();`而`bc`的类型为`ByteChunk`。

4. 继续在**[ByteChunk](https://github.com/apache/tomcat/blob/trunk/java/org/apache/tomcat/util/buf/ByteChunk.java)**搜索`toStringInternal()`方法，其代码如下
```java
public String toStringInternal() {
    if (charset == null) {
        charset = DEFAULT_CHARSET;
    }
    // new String(byte[], int, int, Charset) takes a defensive copy of the
    // entire byte array. This is expensive if only a small subset of the
    // bytes will be used. The code below is from Apache Harmony.
    CharBuffer cb = charset.decode(ByteBuffer.wrap(buff, start, end - start));
    return new String(cb.array(), cb.arrayOffset(), cb.length());
}
```
查看该方法可知其使用`new String(cb.array(), cb.arrayOffset(), cb.length())`的方式来构造`String`对象，故利用`==`比较字符串时其返回值为false。

分析了最基本的Servelt后，由于**[SpringMVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#spring-web)**是基于Servlet实现的，故使用如下代码进行参数比较其值也为false。
```java
@RequestMapping("addUser")
public String addUser(UserModel user) {
	System.out.println(user.getUsername() == "Rosen");
	return StringConstant.SUCCESS;
}
```

参考文章:

* [轻松看懂Java字节码](https://juejin.im/post/5aca2c366fb9a028c97a5609)
* [The Java® Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se8/html)