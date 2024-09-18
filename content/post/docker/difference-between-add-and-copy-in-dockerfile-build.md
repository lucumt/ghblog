---
title: "Dockerfile中ADD与COPY的使用异同"
date: 2021-01-16T15:00:05+08:00
lastmod: 2021-01-16T15:00:05+08:00
draft: false
keywords: ["docker","dockerfile","ADD","COPY"]
description: "简要记录Dockerfile中ADD与COPY的使用异同"
tags: ["docker"]
categories: ["容器化"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
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

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

简要说明在使用[**Dockerfile**](https://docs.docker.com/reference/dockerfile/)构建镜像时，[**ADD**](https://docs.docker.com/reference/dockerfile/#add)与[**COPY**](https://docs.docker.com/reference/dockerfile/#copy)指令的使用差异。

<!--more-->

近期在使用`Docker`时，发现部分项目中采用`Dockerfile`来构建自定义镜像时存在`COPY`和`ADD`两条指令混用的情况，为了规范使用，基于网络上的相关资料，简要的对比总结了它们的差异，供后续参考。

## 对比

在`Docker`官网关于`COPY`指令的[描述](https://docs.docker.com/reference/dockerfile/#copy)如下

> The `COPY` instruction copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.

关于`ADD`的[描述](https://docs.docker.com/reference/dockerfile/#add)如下

> The `ADD` instruction copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.

从它们的描述可知，这两条指令的作用基本上类似，都是将特定文件或文件夹拷贝到镜像中，不同的是`ADD`指令还能从远程URL中拷贝文件并添加到镜像中。

继续查看对应的文档，在`ADD`的指令描述下有如下说明

> If `<src>` is a local `tar` archive in a recognized compression format (`identity`, `gzip`, `bzip2` or `xz`) then it's unpacked as a directory. Resources from remote URLs aren't decompressed. When a directory is copied or unpacked, it has the same behavior as `tar -x`.

从中可知若要拷贝的本地文件属于`tar`系列的压缩文件，则会自动解压为对应的文件夹或文件，这是其相对于`COPY`的另一个显著的不同点。

关于它们更直观的对比说明，可在[源文件](https://github.com/moby/moby/blob/22.06/builder/dockerfile/dispatchers.go#L111)中查看相应的备注:

```go
// COPY foo /path
//
// Same as 'ADD' but without the tar and remote url handling.
func dispatchCopy(d dispatchRequest, c *instructions.CopyCommand) error {
    // xxx
}
```

总结下来就是 **`COPY`指令处理不能进行压缩文件和URL文件流的处理之外，其它功能与`ADD`指令类似。**

## 验证

为了便于观察`Dockerfile`构建过程中的输出，可在构建指令中加入`--progress=plain`来查看输出的详细信息，类似如下

```dockerfile
docker build --progress=plain --no-cache -t custom:v1.0 -f Dockerfile .
```

### ADD指令

* 验证普通的文件拷贝

  `Dockerfile`文件如下

  ```dockerfile
  FROM ubuntu:18.04
  RUN mkdir /home/lucumt
  WORKDIR /home/lucumt
  ADD test.sh /home/lucumt
  RUN ls /home/lucumt
  ```

  输出结果如下

  ![ADD指令使用普通文件](/blog_img/docker/difference-between-add-and-copy-in-dockerfile-build/add-with-ordinary-file.png "ADD指令使用普通文件") 

* 验证URL文件流下载

  `Dockerfile`文件如下[^1]

  ```dockerfile
  FROM ubuntu:18.04
  RUN mkdir /home/lucumt
  WORKDIR /home/lucumt
  ADD https://nginx.org/download/nginx-1.0.15.tar.gz /home/lucumt
  RUN ls /home/lucumt
  ```

  输出结果如下，从中可以看出对于URL类型的文件，在`Dockerfile`的构建过程中只会拷贝，不会解压缩

  ![ADD指令使用URL文件](/blog_img/docker/difference-between-add-and-copy-in-dockerfile-build/add-with-url-file.png "ADD指令使用URL文件") 

* 验证tar文件解压

  `Dockerfile`文件如下

  ```bash
  FROM ubuntu:18.04
  RUN mkdir /home/lucumt
  WORKDIR /home/lucumt
  ADD nginx-1.0.15.tar.gz /home/lucumt
  RUN ls /home/lucumt
  ```

  输出结果如下，可以看出对于从本地拷贝的`tar`类型文件，`Dockerfile`构建过程中会默认进行解压缩

  ![ADD指令使用tar文件](/blog_img/docker/difference-between-add-and-copy-in-dockerfile-build/add-with-tar-file.png "ADD指令使用tar文件") 

### COPY指令

* 验证普通文件

  `Dockerfile`文件内容如下

  ```dockerfile
  FROM ubuntu:18.04
  RUN mkdir /home/lucumt
  WORKDIR /home/lucumt
  COPY test.sh /home/lucumt
  RUN ls /home/lucumt
  ```

  输出结果如下

  ![COPY指令使用普通文件](/blog_img/docker/difference-between-add-and-copy-in-dockerfile-build/copy-with-ordinary-file.png "COPY指令使用普通文件") 

* 验证URL文件流下载

  `Dockerfile`文件内容如下

  ```dockerfile
  FROM ubuntu:18.04
  RUN mkdir /home/lucumt
  WORKDIR /home/lucumt
  COPY https://nginx.org/download/nginx-1.0.15.tar.gz /home/lucumt
  RUN ls /home/lucumt
  ```

  输出结果如下，可看出由于`COPY`指令不支持URL格式，构建过程会出错

  ![COPY指令使用URL文件](/blog_img/docker/difference-between-add-and-copy-in-dockerfile-build/copy-with-url-file.png "COPY指令使用URL文件") 

* 验证tar文件解压

  `Dockerfile`文件内容如下

  ```dockerfile
  FROM ubuntu:18.04
  RUN mkdir /home/lucumt
  WORKDIR /home/lucumt
  COPY nginx-1.0.15.tar.gz /home/lucumt
  RUN ls /home/lucumt
  ```

  输出结果如下，可看出此时对于本地压缩文件不会进行解压。

  ![COPY指令使用压缩文件](/blog_img/docker/difference-between-add-and-copy-in-dockerfile-build/copy-with-tar-file.png "COPY指令使用压缩文件") 

## 总结

官方推荐的是**在不需要使用`ADD`指令的高级特性的场景下，优先使用`COPY`指令，其更直观也不会造成困惑**。

假设对于一个新手来说，采用类似`ADD nginx-1.0.15.tar.gz /home/lucumt`进行构建后最后却得到的是一个解压文件，在不熟悉相关指令细节时会让人觉得很奇怪。

参考链接：

1. [ADD or COPY](https://docs.docker.com/build/building/best-practices/#add-or-copy)
2. [Docker ADD vs. COPY: What are the Differences?](https://phoenixnap.com/kb/docker-add-vs-copy)
3. [What is the difference between the COPY and ADD commands in a Dockerfile?](https://stackoverflow.com/questions/24958140/what-is-the-difference-between-the-copy-and-add-commands-in-a-dockerfile)

[^1]: 从[1.6.0](https://docs.docker.com/build/dockerfile/release-notes/#160) 版本开始，`ADD`指令支持通`--checksum`属性对远程URL文件在下载时进行校验来确保文件准确