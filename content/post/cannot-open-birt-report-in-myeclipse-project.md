+++
author = "飞狐"
categories = ["Web编程","工具使用"]
tags = ["Birt"]
date = "2017-04-07T10:47:28+08:00"
description = "Blog of Rosen Lu"
keywords = ["Birt"]
title = "在MyEclipse项目中不能打开birt报表的解决方法"

+++

由于业务要求，需要在利用MyEclipse中开发的Web项目中添加`Birt`报表统计功能，新建完一个 *report.rptdesign* 文件后双击该文件出现如下错误:  
![无法打开报表文件](/blog_img/cannot-open-birt-report-in-myeclipse-project/cannot-open-birt.png "无法打开birt报表文件")  
错误信息提示MyEclipse无法打开 *Birt* 报表编辑器，对于此种情况，网上搜索相应的解决方案后一般都让我们给该项目添加报表支持，即选中该项目，然后右键 *MyEclipse* -> *Add Report  Capabilities* 来对该项目添加 *Birt* 报表支持。此种方式虽能解决问题，但却同时额外的引入了系统内置的 *Birt* jar包和相应的report文件夹，给我们的项目造成了一定的干扰。  
![birt项目](/blog_img/cannot-open-birt-report-in-myeclipse-project/birt-project-in-myeclipse.png "MyEclipse中的birt项目")

分析上述添加完 *Birt* 报表支持的项目文件后，可发现与普通的Java Web项目相比  *.project* 文件在 *natures* 下面多了一个  *com.genuitec.eclipse.reporting.reportnature* 的配置项，而该配置项正是用于在项目中支持 *Birt* 报表操作。

据此，我们可以采用另外一个方式来解决在MyEclipse中无法打开 *Birt* 报表的问题：**将 *com.genuitec.eclipse.reporting.reportnature* 配置项添加到当前项目 *.project* 文件的 *natures* 配置项下面**，如下所示:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<projectDescription>
	<name>teq</name>
    <comment></comment>
    <projects>
    </projects>
    <buildSpec>
           ...
    </buildSpec>
    <natures>
        <!--添加报表支持 -->
        <nature>com.genuitec.eclipse.reporting.reportnature</nature>
        <nature>com.genuitec.eclipse.j2eedt.core.webnature</nature>
        <nature>org.eclipse.jdt.core.javanature</nature>
        <nature>org.eclipse.wst.jsdt.core.jsNature</nature>
    </natures>
</projectDescription>
```
利用此种方式既可解决无法打开 *Birt* 报表问题，又能避免添加冗余的jar文件和文件夹，给我们的开发省去不必要的麻烦。