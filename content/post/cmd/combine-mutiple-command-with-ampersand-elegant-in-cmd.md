---
title: "在批处理脚本中利用&&命令将多条命令优雅的拼接在一起"
date: 2023-03-10T22:48:56+08:00
lastmod: 2023-03-10T22:48:56+08:00
draft: false
keywords: ["bat","&&","批处理"]
description: "在批处理脚本中利用&&命令将多条命令优雅的拼接在一起，在实现功能的同时保持脚本代码的格式整洁与可维护性"
tags: ["cmd"]
categories: ["脚本操作"]
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
  enable: true
  options: "{
              'x': 0,
              'y': 0,
              'width':1,
              'line-width': 1
            }"

sequenceDiagrams: 
  enable: false
  options: ""

---

简要介绍在`Windows`系统中采用基于`CMD`的批处理脚本进行自动化部署时，对通过`&&`拼接的指令以更优雅的方式实现，以提高批处理脚本的可读性和可维护性。

<!--more-->

## 问题说明

### 项目背景

公司有个基于`Python`开发的项目近期要交付给客户，为了便于安装和后续升级，之前都是采用[PyInstaller](https://pyinstaller.org/)安装成exe可执行文件实现傻瓜式的操作，而这个项目出于避免与客户服务器其它`Python`环境冲突以及便于后续扩展的考量，在软件部署和升级时采用了[Anaconda](https://www.anaconda.com/)进行隔离式部署，其部署流程如下[^1]

```flow
start=>start: 开始
end=>end: 结束
conda_create=>operation: 创建conda环境
conda_active=>operation: 激活conda环境
pip_install=>operation: pip安装依赖
python_run=>operation: 执行python程序

start->conda_create
conda_create->conda_active
conda_active->pip_install
pip_install->python_run
python_run->end
```

各步骤涉及到的操作主要有:

1. 创建conda环境，`conda create -n lyq_test -y`
2. 激活conda环境，`conda activate lyq_test`
3. 安装pip依赖，`pip install -r requirements.txt`
4. 执行python程序，`python hello.py`

这些步骤对于专业的开发人员而言操作起来很快速，但对于相关的使用客户而言，他们不一定具有相关的编程经验，执行上述指令容易出错，便利性有待提高。

由于目标客户的环境为`Windows`，同项目成员沟通后决定采用`CMD`批处理脚本来包含上述命令，实际部署或升级时只需要运行该批处理脚本即可。

### 面临的问题

最开始采用的install.bat批处理脚本类似如下

```shell
@echo off
title 一键部署与升级脚本
echo ===========开始创建conda环境==============
conda create -n lyq_test -y
conda activate lyq_test
pip install -r requirements.txt
python hello.py
pause
```

运行结果如下，可发现从`conda create -n lyq_test -y`之后的指令均没有执行

![在conda create之后终止执行](/blog_img/cmd/combine-mutiple-command-with-ampersand-elegant-in-cmd/batch-file-terminated-after-conda-create.png "在conda create之后终止执行") 

一开始自己以为是由于`Anaconda`的创建需要时间，于是参考`Stackoverflow`中的说明[^2]在`conda activate`之前添加了一条睡眠指令，测试结果和上图一样，问题依旧。

```sh
@echo off
title 一键部署与升级脚本
echo ===========开始创建conda环境==============
conda create -n lyq_test -y
rem 睡眠10秒钟
ping -n 10 127.0.0.1 > nul
conda activate lyq_test
pip install -r requirements.txt
python hello.py
pause
```

参考网上的资料和自己尝试后发现将批处理脚本中的命令用`&&`[^3]拼接起来然后一起执行就能达到要求

```shell
@echo off
title 一键部署与升级脚本
echo ===========开始创建conda环境==============
conda create -n lyq_test -y && conda activate lyq_test && pip install -r requirements.txt && python hello.py
pause
```

运行结果如下，虽然批处理脚本的执行结果符合自己的预期，但此种解决方案带来了一个很严重的副作用

> **通过`&&`将多条指令拼接在一起的结果是脚本中只有一条超长的指令，会导致批处理脚本的可读性和可维护性都严重降低！**

![批处理脚本正常执行](/blog_img/cmd/combine-mutiple-command-with-ampersand-elegant-in-cmd/batch-file-executed-successfully.png "批处理脚本正常执行") 

## 解决方案

由于随着项目的进行，install.bat脚本肯定会进行不断地修正与更新，install.bat脚本的可维护性与可读性是必须要解决的问题，但采用`&&`拼接的方式也不能放弃，只能另想它法。

继续寻求`Stackoverflow`[^4]的协助[^5]，将install.bat通过`SET`来重新赋值的方式进行变量拼接，修改后的代码如下

```shell
@echo off
title 一键部署与升级脚本
echo ===========开始创建conda环境==============
set command=conda create -n lyq_test -y 
set command=%command% && conda activate lyq_test
set command=%command% && pip install -r requirements.txt
set command=%command% && python hello.py
echo 拼接后的命令如下
echo %command%
echo 开始执行拼接后的命令
%command%
pause
```

执行结果如下，虽然能成功进入`Anaconda`环境，但是后续的指令都没有执行。

![批处理脚本中途执行失败](/blog_img/cmd/combine-mutiple-command-with-ampersand-elegant-in-cmd/batch-file-execute-failed-1.png "批处理脚本中途执行失败") 

看起来似乎是胜利在望，但仔细分析后发现只所以能进入`Anaconda`环境是由于第1个`&&`配置生效，导致批处理文件提前执行，而后续的拼接指令没有机会执行[^6]，还是没法实现最终目的。



为了避免提前执行，尝试用双引号将相关指令封装起来，在拼接完成后统一执行

```shell
@echo off
title 一键部署与升级脚本
echo ===========开始创建conda环境==============
set "command=conda create -n lyq_test -y" 
set "command=%command% && conda activate lyq_test"
set "command=%command% && pip install -r requirements.txt"
set "command=%command% && python hello.py"
echo 拼接后的命令如下
echo %command%
echo 开始执行拼接后的命令
%command%
pause
```

执行结果如下，虽然最终的执行结果符合预期，但很明显打印出来的拼接数据不是我们想要的，不利于后续分析，同时一些`echo`打印指令没有执行，此方案存在瑕疵，实际使用中风险很大。

![批处理脚本最终执行成功但无法调试](/blog_img/cmd/combine-mutiple-command-with-ampersand-elegant-in-cmd/batch-file-execute-success-and-echo-wrong-data.png "批处理脚本最终执行成功但无法调试")

经过一段时间的搜索后没有找到自己想要的答案，只能在`Stackoverflow`上提问希望能获得帮助，最终通过[^7]得到了一个较为满意的解决方案。

该方案主要通过**字符串截取**来实现[^8]，修改后的脚本如下

```shell
@echo off
title 一键部署与升级脚本
echo ===========开始创建conda环境==============
set command=conda create -n lyq_test -y
rem 要特别注意第1次拼接时不能加上:~1,-1
set command="%command% && conda activate lyq_test"
set command="%command:~1,-1% && pip install -r requirements.txt"
set command="%command:~1,-1% && python hello.py"
echo ==========拼接后的命令如下===========
echo %command%
echo ==========开始执行拼接后的命令============
%command:~1,-1%
pause
```

脚本执行结果如下，结果符合预期，在确保脚本能正常执行的前提下也满足了可维护性与可读性，问题解决!

![通过字符串截取方式优化脚本](/blog_img/cmd/combine-mutiple-command-with-ampersand-elegant-in-cmd/batch-file-extracted-command-executed-sucesss.png "通过字符串截取方式优化脚本") 



[^1]: 此处出于简化篇幅的考虑，只列出了这4个步骤，实际项目中的步骤会更多
[^2]: https://stackoverflow.com/questions/735285/how-to-wait-in-a-batch-script
[^3]:`&&`在批处理文件中表示只有当前面的指令执行成功时才执行后面的指令，`&`则表示无论前述指令是否执行成功，都执行后面的指令
[^4]:https://stackoverflow.com/questions/17743757/how-to-concatenate-strings-in-windows-batch-file-for-loop
[^5]:或许求助[ChatGPT](https://chat.openai.com/auth/login)是一个更好的方案
[^6]: 此处是基于个人理解的说明，不一定正确，主要是为了说明通过常规的变量重新赋值方式不可行
[^7]:https://stackoverflow.com/questions/75646525/how-to-use-in-command-line-in-variable-in-windows
[^8]:https://superuser.com/questions/228794/how-to-extract-part-of-a-string-in-windows-batch-file



