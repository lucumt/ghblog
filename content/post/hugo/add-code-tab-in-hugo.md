---
title: "给Hugo博客添加代码分组"
date: 2024-11-01T21:13:04+08:00
lastmod: 2024-11-01T21:13:04+08:00
draft: true
keywords: ["hugo"]
description: "简要记录如何给Hugo博客添加代码分组，以便在代码较多时能减少页面篇幅同时便于阅读"
tags: ["hugo","Go"]
categories: ["个人博客"]
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

<!--more-->

 {{% codetabs %}}   

```javascript
import * as React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(<App />, window.document.getElementById('root'));
```

```go::Go语言编程
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

{{% /codetabs %}}

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

 {{% codetabs %}}   

```sql
INSERT INTO table2
SELECT * FROM table1
WHERE condition;
```

```php::世界上最好的语言
<?php
echo "My first PHP script!";
?>
```

```python
a = 33
b = 33
if b > a:
  print("b is greater than a")
elif a == b:
  print("a and b are equal")
```

{{% /codetabs %}}