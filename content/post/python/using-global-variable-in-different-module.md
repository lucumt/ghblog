---
title: "在Python的不同模块中使用全局变量"
date: 2023-01-05T10:55:48+08:00
lastmod: 2023-01-05T10:55:48+08:00
draft: false
keywords: ["python","global"]
description: "记录如何在Python的不同模块中使用相同的全局变量，避免踩坑"
tags: ["python"]
categories: ["python编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: false
borderImage: true

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

简要介绍如何在`Python`中利用[**global**](https://docs.python.org/3/reference/simple_stmts.html#global)关键字定义的全局变量在不同模块间共享，避免后续重复犯错。

<!--more-->

## 背景

它们位于同一个模块下：

```bash
$ ls
__pycache__/  app1.py  main.py
```

各自内容分别如下

`app1.py`有一个 `global`修饰的全局变量，通过`set_variables`方法对其值进行修改

```python
MESSAGE = None


def set_variables():
    global MESSAGE
    MESSAGE = "Hello Python"
```

`main.py`主程序，调用上述方法并打印出`MESSAGE`的值

```python
from app1 import set_variables

if __name__ == '__main__':
    set_variables()
    # print(MESSAGE)
```

## 不生效用法

一开始自己想复用`import xxx from xxx` 这种用法，将`main.py`修改为如下

```python
from app1 import set_variables, MESSAGE

if __name__ == '__main__':
    set_variables()
    print(MESSAGE)
```

结果输出值为`None`，没有达到预期的结果。

## 正确用法

直接用`import xxx`实现

```python
import app1

if __name__ == '__main__':
    app1.set_variables()
    print(app1.MESSAGE)
```

或用`import xxx as xxx`给其加上一个别名

```python
import app1 as a

if __name__ == '__main__':
    a.set_variables()
    print(a.MESSAGE)
```

## 原因分析

参见[**Global variable not changing between files in python**](https://stackoverflow.com/questions/49636945/global-variable-not-changing-between-files-in-python)中的大佬回答如下：

> The syntax `from globals.py import *` makes copies of the variables within `globals.py` into your local file. To access the variables themselves without making copies, `import globals` and use the variable directly: `globals.filename`. You no longer need the `global` keyword if you access the variable this way.

