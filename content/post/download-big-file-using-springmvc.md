+++

author = "飞狐"
categories = ["Java编程","Spring系列"]
tags = ["Java","Spring","SpringMVC"]
date = "2016-03-20T16:41:29+08:00"
description = "Blog of Rosen Lu"
keywords = ["Java","SpringMVC"]
title = "利用SpringMVC下载大文件时内存溢出的处理"

+++
文件的上传和下载是Web系统中的一个很普通的功能，实现的方式也有很多种，如利用 **[java.io](https://docs.oracle.com/javase/7/docs/api/java/io/package-summary.html)** 下面的各种IO类自己实现，或者利用 **[Commons IO](https://commons.apache.org/proper/commons-io/)** 包中的 ***FileUtils*** 、 ***IOUtils*** 类中封装好的方法直接调用。由于目前我所开发的系统采用了 **[SpringMVC](http://docs.spring.io/autorepo/docs/spring/3.2.x/spring-framework-reference/html/mvc.html)** 来作为项目的MVC实现，所以很自然的采用 **[SpringMVC](http://docs.spring.io/autorepo/docs/spring/3.2.x/spring-framework-reference/html/mvc.html)**内置的API进行文件的下载，但在实际使用过程中发现其对大文件的下载支持不太好，现把解决方案记录如下：

<!--more-->

```java
@RequestMapping("downloadRequireDocument")
public ResponseEntity<byte[]> downloadRequireDocument(String fileId,String fileName,String fileType,
     HttpServletRequest request) throws IOException{
	String filePath=fileName+fileId+"."+fileType;
    
	HttpHeaders headers=new HttpHeaders();
	headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
	headers.setContentDispositionFormData("attachment",URLEncoder.encode(fileName,"UTF-8")+"."+fileType);
    
	File downloadFile=new File(request.getSession().getServletContext().getRealPath(File.separator)+filePath);
    
	return new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(downloadFile),headers,HttpStatus.CREATED);
}
```
该段代码在下载小文件时可以正常工作，但是当要下载的文件很大时（如几百M或上G），就会发生如下错误：
```java
java.lang.OutOfMemoryError: Java heap space
at org.apache.commons.io.output.ByteArrayOutputStream.toByteArray(ByteArrayOutputStream.java:271)
at org.apache.commons.io.IOUtils.toByteArray(IOUtils.java:219)
at org.apache.commons.io.FileUtils.readFileToByteArray(FileUtils.java:1136)
```
去网上搜索 ***java.lang.OutOfMemoryError: Java heap space*** 这个错误时，一般都建议我们在tomcat中添加如下类似设置来提高JVM的配置:  
```set JAVA_OPTS=%JAVA_OPTS% -server -Xms800m -Xmx800m -XX:MaxNewSize=256m -XX:MaxPermSize=256m```  

但即使按照把上面的参数配置都扩大一倍，在下载更大的文件时还是会遇到 ***java.lang.OutOfMemoryError: Java heap space*** 这个错误，上面的解决方法治标不治本。分析下异常堆栈可以发现问题产生的根源在于 *at org.apache.commons.io.FileUtils.readFileToByteArray(FileUtils.java:1136)* 这行代码，***FileUtils.readFileToByteArray***  会把文件一次性读入内存中，要下载的文件越大，需要占用的内存也越大，当文件的大小超过JVM和Tomcat的内存配置时，***OutOfMemoryError*** 这个问题就会不可避免的发生。

弄清产生该问题的原因之后，解决的方法也很简单：**不利用[Commons IO](https://commons.apache.org/proper/commons-io/)把文件一次性读入内存，而是利用普通的文件输出流按字节分段写入文件，把占用的内存固定在一个指定的范围内，从根本上避免内存占用过高的问题**,替代的代码如下:
```java
@RequestMapping("downloadRequireDocument")
public void downloadRequireDocument(String fileId,String fileName,String fileType,
	HttpServletRequest request,HttpServletResponse response) throws IOException {
	
	String filePath = request.getSession().getServletContext().getRealPath(File.separator)+fileName+"."+fileType;
	fileName = URLEncoder.encode(fileName.trim(),"UTF-8")+"."+fileType;
	response.setHeader("Content-Disposition","attachment;filename="+fileName);

	InputStream is = new FileInputStream(filePath);
	
	int read =0;
	byte[] bytes = new byte[2048];
	OutputStream os = response.getOutputStream();
	while((read = is.read(bytes))!=-1){//按字节逐个写入，避免内存占用过高
		os.write(bytes, 0, read);
	}
	os.flush();
	os.close();
	is.close();
}
```
