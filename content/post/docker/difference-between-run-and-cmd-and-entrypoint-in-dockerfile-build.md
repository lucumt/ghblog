---
title: "Dockerfile中RUN、CMD与ENTRYPOINT的差异比较"
date: 2021-02-20T15:00:08+08:00
lastmod: 2021-02-20T15:00:08+08:00
draft: false
keywords: ["docker","dockerfile","RUN","CMD","ENTRYPOINT"]
description: "通过示例子简要说明Dockerfile中RUN、CMD与ENTRYPOINT的差异比较，为Dockerfile的编写提供参考"
tags: ["docker"]
categories: ["容器化","翻译"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
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

本文翻译自[**Docker RUN vs CMD vs ENTRYPOINT**](https://codewithyury.com/docker-run-vs-cmd-vs-entrypoint/)。

<!--more-->

`Dockerfile`使用时的`CMD`与`ENTRYPOINT`指令其作用很相似，经常让人容易混淆，网络上有不少关于它们对比说明的文章，但个人感觉都不直观明晰。

本来自己想专门写一篇关于它们差异的博文，后来在网上发现国外有个作者已经写了类似的文章，同时读了之后有醍醐灌顶之感，故出于对作者的尊重与感谢，决定直接将其翻译成中文！

<!-正文开始 ->

`Docker`中的一些指令看起来很相似，它们容易导致初学者的困惑和不恰当的使用，本文将以具体示例来说明`CMD`、`RUN`和`ENTRYPOINT`这3者的差异。

## 概要说明

* `RUN`用于在一个镜像层中执行指令并创建一个新的镜像，如通常用来安装软件包
* `CMD`用于设置默认的命令或参数，它们可以在容器运行时通过命令行覆盖
* `ENTRYPOINT`用于将容器设置为通过可执行文件来运行[^1]

如果还是不太明白或者想了解更详细的信息，请继续往下阅读。

## Docker镜像与分层

`Docker`运行一个容器时实际上是运行容器里面的镜像，这些镜像通常是通过`Docker`指令构建的，这些指令在已有的镜像层或操作系统发行版的顶层添加新的分层，操作系统发行版是一个初始镜像，所有在其上添加的分层都会创建一个新的镜像。



最终的`Docker`镜像就像一个洋葱，里面包含由操作系统发行版以及其上的许多镜像分层，例如可通过基于Ubuntu 14.04发行版安装一系列的deb软件包来构建自己的镜像。

## Shell和Exec模式

这3条指令(`RUN`、`CMD`、`ENTRYPOINT`)都可基于`shell模式`和`exec模式`运行，由于这些模式相对于指令更容易让人困惑，我们先熟悉下这些指令。

### Shell模式

`<instruction> <command>`

示例:

```dockerfile
RUN apt-get install python3
CMD echo "Hello world"
ENTRYPOINT echo "Hello world"
```

当以`shell模式`执行指令时，会在后台调用`/bin/sh -c <command>`并进行正常的shell处理，以下述`Dockerfile`中的代码片段为例

```dockerfile
ENV name John Dow
ENTRYPOINT echo "Hello, $name"
```

当以`docker run -it <image>`的形式运行容器时，会有如下输出

```bash
Hello, John Dow
```

可以看出在输出结果中变量name已经被其实际值替换了。

### Exec模式

此模式是`CMD`和`ENTRYPOINT`指令说明中推荐的模式。

`<instruction> ["executable", "param1", "param2", ...]`

示例:

```dockerfile
RUN ["apt-get", "install", "python3"]
CMD ["/bin/echo", "Hello world"]
ENTRYPOINT ["/bin/echo", "Hello world"]
```

当指令以`exec模式`执行时，其会直接调用可执行文件，同时shell处理不会生效，以下述`Dockerfile`中的代码片段为例：

```dockerfile
ENV name John Dow
ENTRYPOINT ["/bin/echo", "Hello, $name"]
```

当以`docker run -it <image>`的形式运行容器时，会有如下输出

```
Hello, $name
```

可看出输出结果中的name变量并没有被解析替换。

### 运行bash脚本

如果需要运行`bash`脚本(或其它的shell脚本)，可通过在`exec模式`中使用`/bin/bash`来将其设置为可运行，在这种方式下，常规的shell处理会生效，以下述`Dockerfile`中的代码片段为例：

```dockerfile
ENV name John Dow
ENTRYPOINT ["/bin/bash", "-c", "echo Hello, $name"]
```

当以`docker run -it <image>`的形式运行容器时，会有如下输出

```bash
Hello, John Dow
```

## RUN指令

`RUN`指令允许我们在已有的镜像中安装所需的程序和软件包，它在当前镜像的顶层执行相关指令并通过提交执行结果的方式创建一个新的镜像封层，通常可在一个`Dockerfile`中看见多个`RUN`指令。



`RUN`指令有2种使用方式:

* `RUN <command>` (`shell模式`)
* `RUN ["executable", "param1", "param2"]` (`exec模式`)

(这两种模式在前面已有详细的说明)



`RUN`指令一个典型的使用场景为安装多个版本控制软件包

```bash
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

需要注意的是`apt-get update`和`apt-get install`是在一条指令中执行，这样操作是为了确保安装最新的软件包。如果`apt-get install`是在单独的指令中执行，则其会复用`apt-get update`生成的镜像层，而该镜像层可能会在很久之前就被创建[^2]。

## CMD指令

`CMD`指令允许我们设置一个默认的命令，当我们在运行容器时没有显示指定要运行的命令，默认命令将会执行，若`Docker`容器运行时指定了命令，则其将会被忽略。如果一个`Dockerfile`中有多个`CMD`指令，除了最后一个之外的其余`CMD`指令都会被忽略。



`CMD`指令有3种使用方式：

* `CMD ["executable","param1","param2"]` （`exec模式`，首选）
* `CMD ["param1","param2"]` （在`exec模式`下给`ENTRYPOINT`指令添加额外的参数）
* `CMD command param1 param2` (`shell`模式)

再次说明，前面已对第1种和第3种方式进行了解释，第2种方式则是在`exec模式`下和`ENTRYPOINT`组合使用，如果容器在没有命令行参数的情况下运行，它会设置将在`ENTRYPOINT`指令之后附加的默认参数。具体请查看`ENTRYPOINT`相关的说明。



我们以下述`Dockerfile`中的代码片段为例，来看下`CMD`指令是如何运行的

```dockerfile
CMD echo "Hello world" 
```

当以`docker run -it <image>`的形式运行容器时，会有如下输出

```bash
Hello world
```

但若运行容器时有附带的命令，如`docker run -it <image> /bin/bash`，此时`CMD`指令会被忽略，默认的`bash`会执行

```bash
root@7de4bed89922:/#
```

## ENTRYPOINT指令

`ENTRYPOINT`用于将容器设置为通过可执行文件来运行，由于它也能通过设置附带参数的命令，其看起来和`CMD`指令很相似，不同之处为`ENTRYPOINT`命令和参数在`Docker`容器以带参数的命令运行时不会被忽略。(虽然有办法来让`Docker`容器忽略`ENTRYPOINT`命令和参数，但一般我们不会这么做)



`ENTRYPOINT`指令有2种使用方式:

* `ENTRYPOINT ["executable", "param1", "param2"]` (`exec模式`，首选)
* `ENTRYPOINT command param1 param2` (`shell模式`)

当使用`ENTRYPOINT`模式时需要非常小心，这是由于不同方式的行为差异很大。

### Exec模式

`ENTRYPOINT`指令中的`Exec模式`允许我们设置命令和参数，然后使用任一形式的`CMD`模式来设置有可能有可能发生变化的附带参数。`ENTRYPOINT`指令中的参数会一直被用到，而`CMD`中的相关参数则可在`Docker`容器运行覆盖，以下述`Dockerfile`中的代码片段为例

```dockerfile
ENTRYPOINT ["/bin/echo", "Hello"]
CMD ["world"]
```

当以`docker run -it <image>`的形式运行容器时，输出如下

```bash
Hello world
```

但当`Docker`容器以`docker run -it <image> John`的形式运行时，输出如下

```bash
Hello John
```

### Shell模式

`ENTRYPOINT`指令中的`shell模式`会忽略任何`CMD`或`Docker`容器运行时附带的参数。

## 总结

使用`RUN`指令通过在初始镜像的顶层添加分层来创建我们自己的镜像。



当构建镜像且需要相关命令始终被执行时，优先使用`ENTRYPOINT`而非`CMD`指令，如果有些参数在`Docker`容器运行期间会被覆盖/改变，可通过额外使用`CMD`指令来添加默认的参数。



当需要添加在`Docker`容器运行期间可被覆盖的默认参数，可使用`CMD`指令。

[^1]: 译注: 即让容器一直保存运行状态，如通过`java -jar springboot.jar`让程序一直运行
[^2]: 译注： 原因为通过`Dockerfile`构建镜像时缓存默认生效