---
title: "利用Numpy将npz文件解析为csv文件"
date: 2023-05-16T10:09:16+08:00
lastmod: 2023-05-16T10:09:16+08:00
draft: false
keywords: ["python","numpy","npz","csv"]
description: "简要记录在Python编程中如何利用Numpy将npz文件解析为csv文件-供参考"
tags: ["python"]
categories: ["数据计算"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
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

记录下如何基于[**NumPy**](https://numpy.org/)将`npz`文件解析成多个`csv`文件，以自己项目中使用的`npz`文件结构为例，其它地方使用时可能会有差异 

<!--more-->

执行下述代码分析其key结构

```python
if __name__ == '__main__':
    src_file = r'D:\test-001.npz'
    results = np.load(src_file)
    for key, value in results.items():
        print(key)
```

输出结果类似如下

```bash
CAN/2/localmap1__message394/left_line_id_11
CAN/2/localmap1__message394/left_line_id_12
CAN/2/localmap1__message394/left_line_id_13
CAN/2/localmap1__message396/_timestamp
CAN/2/localmap1__message396/timeStamp
CAN/2/localmap1__message396/right_line_id_15
```

基于此可发现其组织结构为`协议`/`通道`/`序号`/`信号`/`报文`，我们要做的时基于以信号为单位生成`csv`文件

改进测试代码

```python
if __name__ == '__main__':
    src_file = r'D:\test-001.npz'
    results = np.load(src_file)
    for key, value in results.items():
        print(key + "\t" + str(len(value)))
```

执行后输出结果类似如下

```bash
CAN/2/localmap1__message394/left_line_id_11	322
CAN/2/localmap1__message394/left_line_id_12	322
CAN/2/localmap1__message394/left_line_id_13	322
CAN/2/localmap1__message396/_timestamp	350
CAN/2/localmap1__message396/timeStamp	350
CAN/2/localmap1__message396/right_line_id_15	350
```

可发现一个报文对应多行记录，且每个信号下的报文记录数基本上相同。

同时也能发现在`npz`中的数据是一行一行的，逐行拼接在一起，若要转化为`csv`文件，则涉及到行专列操作，同样可基于`NumPy`进行操作，完整的代码如下

```python
import os
import shutil
import numpy as np
import time
import sys

def parse_npz(data_folder, src_file):
    if os.path.exists(data_folder):
        shutil.rmtree(data_folder)
    os.makedirs(data_folder)

    results = np.load(src_file)
    dict = {}
    message_name = None
    for key, value in results.items():
        data = key.split("/")

        # 长度不符合要求，直接跳过
        if (len(data)) != 4:
            continue
        new_message = data[-2]

        # 新的信号开始，要处理之前的信号
        if new_message != message_name:
            process_message(data_folder, message_name, dict)
            message_name = new_message
            dict[message_name] = []
        signal = str(data[-1])
        is_timestamp = signal == '_timestamp'
        if is_timestamp:
            value = np.multiply(value, 100_0000).astype(np.int64)
            value = value - value[0]
            signal = 'Timestamp'
        dict[message_name].append([signal, *value])

    # 处理最后一个信号
    process_message(data_folder, message_name, dict)


def process_message(folder, message, dict):
    if message is None:
        return
    signals = dict[message]
    length = min(len(v) for v in signals)
    signals = [v[0:length] for v in signals]

    # 将行转化为列，以符合业务需求
    csv_data = np.array(signals).swapaxes(0, 1)

    # 保存到磁盘
    file_name = folder + os.sep + message + ".csv"
    np.savetxt(file_name, csv_data, delimiter=",", fmt="%s")
    # print('-------------save file to ' + file_name + '---------------')
    del dict[message]


if __name__ == '__main__':
    #data_folder = sys.argv[1]
    #src_file = sys.argv[2]
    data_folder = r'D:\npz_csv'
    src_file = r'D:\test-001.npz'
    start = time.time()
    parse_npz(data_folder, src_file)
    end = time.time()
    print(f'=============== Parse npz file time cost:{end - start:.4f}s =============')
```