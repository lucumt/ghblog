+++
author = "飞狐"
categories = ["Go编程","工具使用"]
tags = ["Go","Intellij IDEA"]
date = "2017-03-05T14:04:43+08:00"
description = "Blog of Rosen Lu"
keywords = ["Go","Intellij IDEA"]
title = "在Intellij IDEA中引用Golang本地包"

+++

在学习`Golang`时，自己最开始用的是eclpse中的[goclipse](https://goclipse.github.io/)插件来进行 *Golang* 编程，但其对*Golang* 的支持不是太好，如代码格式化、自动导入引用包等都无法直接在eclipse中使用，并且其自动提示功能也没有像 *Java* 那么强，于是转用[Intellij IDEA](https://www.jetbrains.com/idea/)安装[Golang插件](https://plugins.jetbrains.com/plugin/5047-go)来替代使用，安装完插件后的[Intellij IDEA](https://www.jetbrains.com/idea/)对 *Golang* 的支持在各方面都很令人满意，唯独引入本地包的支持不太好用。经过一阵摸索自己找出了解决方案，先记录下。

<!--more-->

## 在Eclipse中引用Golang本地包
若我们采用的是[goclipse](https://goclipse.github.io/)来开发 *Golang* ,则在其中引用本地包很简单，和引用Java包类似。如下图所示，假设 *src* 是源代码所在的目录，在 *src* 的 *sec* 文件夹下有一个名为 *calculate.go* 的文件，其中有一个名为 *Add* 的函数用于计算两个整数的相加之和。  
![goclipse中的本地包](/blog_img/import-local-page-in-intellij-idea/goclipse_package.png)  
若要在主程序main方法中调用 *Add* 方法，先通过import引入该文件的包名`import service`，然后通过包名调用该方法`service.Add(1,2)`，如下图所示，可以看出在Eclipse中引用 *Golang* 本地包与引用Java包没有太大的区别，都是将包文件放到 *src* 源文件夹下，然后通过包名来引用。  
![goclipse中的本地包引用](/blog_img/import-local-page-in-intellij-idea/goclipse_package_reference.png)  
也可以通过直接导入该包所在的文件夹的名称来调用该方法，此时需要将`import service`改为`import sec`，如下图所示  
![goclipse中的本地包通过文件夹引用](/blog_img/import-local-page-in-intellij-idea/goclipse_package_reference_folder.png)  
可以看出在eclipse中导入 *Golang* 本地包时有两种方法： **通过import导入包名** 或 **通过import导入该包对应的文件夹** ，这两种方法均可使程序正常运行。

## 在Intellij IDEA中引用Golang本地包
下面2张图为在IDEA中建立的对应项目，图中 *gproject* 是一个项目，*gotest* 是一个模块，我们在 *gotest* 下建立相关的测试文件。  
![IDEA中的本地包](/blog_img/import-local-page-in-intellij-idea/idea_package.png)    
![IDEA中的本地包引用](/blog_img/import-local-page-in-intellij-idea/idea_package_reference.png)  
在上述代码中我们是通过`import service`的方式来导入相应包的，由于IDEA对Golang很强，从图中可以看出 *service* 的颜色与其它导入包的颜色不一致，当把鼠标移动到 *service* 上时会提示 *Cannot resolve file 'service'* ，直接运行时，会出现如下图所示 *cannot find package* 错误，将`import service`修改为`import sec`时，会出现同样的错误，可以看出在idea中默认不支持直接导入本地 *Golang* 包。       
![IDEA中引用本地包运行出错](/blog_img/import-local-page-in-intellij-idea/idea_package_reference_run_error.png)     
从报错信息可以看出，程序在运行时先去 **GOROOT** 去搜索导入包，然后去 **GOPATH** 寻找导入包，最后在当前项目模块下寻找导入包，但实际上不存在 *D:\program\IntelliJ IDEA 2016.2.1\workspace\gproject\gotest\src\service* 这个目录，故而程序报错，不能正常运行。

解决该问题的关键是明白 **[GOROOT](http://golang.org/doc/install#tarball_non_standard)** 和 **[GOPATH](http://golang.org/cmd/go/#hdr-GOPATH_environment_variable)** 的作用，根据官方文档的解释 **GOPATH** 的主要作用是存放文件以便 *Golang* 程序编译时可以进行搜索引用，**GOPATH** 可以设置一个值或多个值，多个值之间以分号隔开。很明显只要我们将本地 *Golang* 加入到 **GOPATH**中即可在IDEA中正常运行该程序。

如下图所示，在IDEA中依次选择 **File->Settings->Language&Frameworks->Go->Go Libraries** ，会出现如下图所示的配置 *Golang* 库的界面，在该界面可以添加 *Golang* 本地包所在的路径，该界面包含3个不同作用范围的配置方式： *Global libraries* 、 *Project libraries* 和 *Module libraries* ，其中 *Global libraries* 的配置对所有项目生效为全局配置，*Project libraries* 的配置对整个项目生效，*Module libraries* 的配置只对模块生效，可以看出在 *Global libraries* 默认包含了 **GOPATH** 。根据实际使用的需求我们可以选择把本地包设置在 *Project libraries* 还是 *Module libraries* 中。  
![IDEA中配置程序库](/blog_img/import-local-page-in-intellij-idea/idea_package_select_gopath.png)  
本文的程序都是在 *gotest* 模块下，故将其添加到 *Module libraries* 下，添加完的结果如下所示：  
![IDEA中配置程序库的结果](/blog_img/import-local-page-in-intellij-idea/idea_package_gopath_config.png)  
将本地包添加到模块库之后，还需要在go文件中将导入包的语句设置为`import sec`，不能设置为`import service`，然后该 *Golang* 程序即可正常运行。

可以看出，不同于[goclipse](https://goclipse.github.io/)，在IDEA中只能使用 **通过import导入该包对应的文件夹** 来导入本地 *Golang* 包，至于原因还需要进一步研究。

## 利用Goclipse时无法运行程序的解决方法
在使用[goclipse](https://goclipse.github.io/)运行 *Golang* 程序时，偶尔会出现程序无法编译和运行的情况，这种情形一般都是 *src* 没有被设置成源代码目录造成的，此时可以通过如下图所示的方法，将 *src* 目录添加源代码目录。在Eclipse中选中该项目然后点击 **Properties** ，会出现项目属性配置界面，点击 **Go Project Configuration** ，通过 **Add Folder** 可以将 *src* 添加到源代码中，之后程序即可正常运行。  
![goclipse中添加源程序目录](/blog_img/import-local-page-in-intellij-idea/goclipse_add_source_folder.png)  