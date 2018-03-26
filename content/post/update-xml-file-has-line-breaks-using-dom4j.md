+++
author = "飞狐"
tags = ["Java","XML"]
categories = ["Java编程"]
date = "2018-01-04T11:21:42+08:00"
description = "Blog of Rosen Lu"
keywords = ["Java","XML"]
title = "利用dom4j修改含有回车换行符的XML文件"

+++

这几天工作中遇到一个利用 **[dom4j](https://dom4j.github.io/)** 更新XML文件的任务，由于XML文件中部分属性包含有换行符，利用 *dom4j(1.6.1)* 默认的方法更新XML文件后换行符会丢失。 各种Google、StackOverflow折腾好久后终于解决该问题，简单记录下。

<!--more-->

对于修改XML文件，自己很自然的想到利用 *dom4j* 和 **[XPath](https://en.wikipedia.org/wiki/XPath)** 来实现功能，使用的代码类似如下：

```java
public static void updateXML() {
	SAXReader saxReader = new SAXReader();
	File oldFile = new File("D:\\test\\old_test.xml");
	File newFile = new File("D:\\test\\new_test.xml");
	try {
		Document oldDoc = saxReader.read(oldFile);
		Element oldRoot = oldDoc.getRootElement();
		Element oldEle = (Element) oldRoot.selectSingleNode("//book[@id='01001']/name");
		System.out.println(oldEle.attributeValue("description"));
		
		OutputFormat format = OutputFormat.createPrettyPrint();
		format.setEncoding("UTF-8");
		format.setNewLineAfterDeclaration(false);
		
		//写入新文件
		XMLWriter writer = new XMLWriter(new FileWriter(newFile), format);
		writer.write(oldDoc);
		writer.flush();
		writer.close();

		System.out.println("\n================分隔符==================\n");
		//从新文件中读取数据
		Document newDoc = saxReader.read(newFile);
		Element newRoot = newDoc.getRootElement();
		Element newEle = (Element) newRoot.selectSingleNode("//book[@id='01001']/name");
		System.out.println(newEle.attributeValue("description"));
	} catch (DocumentException e) {
		e.printStackTrace();
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```    
对应的XML文件类似如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<books>
  <book id="01001">
    <name description="New coverage includes:&#xD;&#xA;&#xD;&#xA;Functional interfaces, lambda expressions, method references, and streams&#xD;&#xA;Default and static methods in interfaces&#xD;&#xA;Type inference, including the diamond operator for generic types&#xD;&#xA;The @SafeVarargs annotation&#xD;&#xA;The try-with-resources statement">Effective Java</name>
    <price>50.6</price>
    <author>Joshua Bloch</author>
    <publishDate>2017-12-08</publishDate>
  </book>
</books>
```
上述代码的逻辑很简单：首先从XML打开一个XML文件，然后输出某个书本的描述信息，将其写入新的XML文件，然后在新XML文件中读取相同的信息。理论上前后两次输出的结果应该一样，但实际运行后发现从新的XML文件中读取出的描述信息换行符都丢失了，前后两次输出的结果不一致！  
!["新旧XML文件的输出格式不同"](/blog_img/update-xml-file-has-line-breaks-using-dom4j/dom4j_incorrect_output.png "新旧XML文件的输入格式不同") 

对比新旧XML文件后发现，产生此现象的原因是: **dom4j 自作主张的在写入XML文件时将 *\&#xD;\&#xA;* 替换为了 *\r\n* ，而XML文件中标准的回车换行符是用 *\&#xD;\&#xA;* 来表示的** ，将它们替换后再次读取的结果很显然不符合要求。

了解到问题产生的根源后，则其解决思路也很明确： **写入XML文件时，将 *\r\n* 再次替换为 *\&#xD;\&#xA;* 即可**。最开始自己想采用如下的方法来简单替换，运行完毕后发现结果和前面的一致，问题依旧。
```java
String description = oldEle.attributeValue("description");
description = description.replaceAll("\r\n","&#xD;&#xA;");
oldEle.attributeValue("description",description);
```
<br/>
进一步分析后发现，*dom4j* 不仅会在读取XML文件时对 *\&#xD;\&#xA;* 进行转义，而且在写入XML文件时也会对 *\&#xD;\&#xA;* 进行转义，前面的方法只是解决了读取的问题，写入时没有处理，所以问题依旧。

写入时主要的操作类是 *OutputFormat* 和 *XMLWriter* ，自己一开始以为可以通过 *OutputFormat* 进行响应的设置实现，将代码修改如下，然并卵，问题依旧！
```java
OutputFormat format = OutputFormat.createPrettyPrint();
format.setNewlines(true);
format.setLineSeparator("\r\n");
format.setEncoding("UTF-8");
format.setNewLineAfterDeclaration(false);
```
<br/>
*OutputFormat* 不好使，只能从 *XMLWriter* 着手，调用 *writer.setEscapeText(false)* 方法也不能解决问题，看来只能放出大招，自己定义实现一个 XMLWriter类，将以及转义后的回车换行符又换回去，代码如下：
```java
public class HRXMLWriter extends XMLWriter {

	public HRXMLWriter(Writer wr, OutputFormat format) {
		super(wr, format);
	}

	@Override
	protected String escapeAttributeEntities(String text) {
		text = super.escapeAttributeEntities(text);
		if (text.indexOf("\r\n") > -1) {
			text = text.replaceAll("\r\n", "&#xD;&#xA;");
		}
		return text;
	}

}
```
然后将写入时的代码修改如下：
```java
//採用定义写入类HRXMLWriter
XMLWriter writer = new HRXMLWriter(new FileWriter(newFile), format);
writer.write(oldDoc);
writer.flush();
writer.close();
```
运行结果如下，问题顺利解决！  
!["新旧XML文件的输出格式相同"](/blog_img/update-xml-file-has-line-breaks-using-dom4j/dom4j_correct_output.png "新旧XML文件的输入格式相同")   

坑爹啊！