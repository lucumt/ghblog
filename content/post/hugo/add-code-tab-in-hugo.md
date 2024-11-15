---
title: "给Hugo博客添加代码分组"
date: 2024-11-01T21:13:04+08:00
lastmod: 2024-11-01T21:13:04+08:00
draft: false
keywords: ["hugo"]
description: "简要记录如何给Hugo博客添加代码分组，以便在代码较多时能减少页面篇幅同时便于阅读"
tags: ["hugo","go"]
categories: ["个人博客"]
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
centerImage: false
borderImage: true
codeTabSeperator: "::"

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

[**Hugo**](https://gohugo.io/)默认没有提供代码分组的支持，本文基于[这条帖子](https://discourse.gohugo.io/t/code-tabs-widget/975)和[gitbook-plugin-prism-codetab-fox](https://github.com/gitbook-plugin-fox/gitbook-plugin-prism-codetab-fox)通过修改[Even](https://github.com/olOwOlo/hugo-theme-even)主题的相关代码，实现基于Tab的代码块分组功能。

<!--more-->

## 代码修改

1. 在`assets/sass/_custom/_custom.scss`中添加如下内容，此部分内容来源于[codetab.css](https://github.com/gitbook-plugin-fox/gitbook-plugin-prism-codetab-fox/blob/master/codetab/codetab.css)

   ```css
   .codetabs {
   	border: 1px solid rgba(211, 220, 228, 1.00);
   }
   .codetabs .codetabs-header {
   	background: #f7f8f9;
   }
   .codetabs .codetabs-header .tab {
   	display: inline-block;
   	padding: 3px 20px;
   	cursor: pointer;
   	opacity: 0.5;
   	border-right: 1px solid rgba(211, 220, 228, 1.00);
   	border-bottom: 1px solid rgba(211, 220, 228, 1.00);
   	background-color: #f7f7f7;
   	font-weight: bold;
   	font-size: 13px;
   }
   .codetabs .codetabs-header .tab.active {
   	border-bottom: 0px;
   	cursor: default;
   	opacity: 1;
   	background: #fff;
   	color: #2674ba;
   }
   .codetabs .codetabs-body .tab {
   	display: none;
   }
   .codetabs .codetabs-body .tab.active {
   	display: block;
   }
   ```

2. 将`layouts/_default/_markup/render-codeblock.html`的内容修改如下,主要是添加了额外的tab头信息用于后续初始化tab并展示

   ```go
   <!-- only show line numbers when there is more than one line or specific it manually -->
   {{- $classStr := "language-%s" }}
   {{- $lineCount := len (split .Inner "\n") -}}
   {{- if gt $lineCount 1 }}
     {{- $classStr = print "line-numbers " print $classStr  }}
   {{- else -}}
     {{- $classStr = print "no-line-numbers " print $classStr  }}
   {{- end }}
   
   {{- $attributes := .Attributes }}
   
   {{- $globalCodeTabSeperator := site.Params.codeTabSeperator | default "#" -}}
   {{- $codeTabSeperator := .Page.Params.codeTabSeperator | default $globalCodeTabSeperator -}}
   {{- $title :=index (split .Type $codeTabSeperator) 1 | default .Type -}}
   {{- $type :=index (split .Type $codeTabSeperator) 0 | default .Type -}}
   
   {{- $classes := slice (printf $classStr $type) .Attributes.class }}
   {{- $attributes = merge $attributes (dict "class" (delimit $classes " ") "title" $title) -}}
   <pre
     {{- range $k, $v := $attributes }}
       {{- printf " %s=%q" $k $v | safeHTMLAttr }}
     {{- end -}}
   >
   <code>
   {{- .Inner -}}
   </code>
   </pre>
   {{- .Page.Store.Set "hasCodeBlock" true }}
   ```

3. 在`layouts/shortcodes`下创建一个名为`codetabs.html`的文件并写入如下内容

   ```html
   <div class='codetabs'>
     <div class="codetabs-header"></div>
     <div class="codetabs-body">{{ .Inner }}</div>
   </div>
   ```

4. 在`layouts/partials/scripts.html`中找到代码块逻辑处理部分

   ```go
   {{ if .Store.Get "hasCodeBlock" }}
     <script src="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/components/prism-core.min.js"></script>
     <script src="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/autoloader/prism-autoloader.min.js"></script>
     <script src="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-highlight/prism-line-highlight.min.js"></script>
     <script src="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/plugins/line-numbers/prism-line-numbers.min.js"></script>
   {{ end }}
   ```

   并在尾部添加如下代码

   ```javascript
   <script type="text/javascript">
        document.addEventListener('DOMContentLoaded', initCodeTabs);
   	 document.addEventListener('DOMContentLoaded', switchTabCodes);
   	 function initCodeTabs(){
   	     document.querySelectorAll('.codetabs').forEach(tab =>{
   		    let tabHeader = tab.querySelector('.codetabs-header');
   			let tabBody = tab.querySelector('.codetabs-body');
   		    let codeblocks = tab.querySelectorAll('.codetabs-body > .highlight-wrapper');
   			codeblocks.forEach((block,index) =>{
   			   let titleContent = block.querySelector('pre').getAttribute('title');
   			   let title = document.createElement('div');
   			   title.classList.add('tab');
   			   if(index == 0){
   			   	  title.classList.add('active');
   			   }
   			   title.setAttribute('data-codetab',index);
   			   title.appendChild(document.createTextNode(titleContent));
   			   tabHeader.appendChild(title);
   			   
   			   let div = document.createElement('div');
   			   div.classList.add('tab');
   			   if(index == 0){
   			   	  div.classList.add('active');
   			   }
   			   div.setAttribute('data-codetab',index);
   			   div.appendChild(block);
   			   tabBody.appendChild(div);
   			});
   		 });
   	 }
   	 function switchTabCodes(){
   		 document.querySelectorAll('.codetabs  .codetabs-header > .tab').forEach(tab => {
   			 tab.addEventListener('click', function() {
   			      if(tab.classList.contains('active')){
   					 return;
   				  }
   				  let tabId = tab.getAttribute('data-codetab');
   				  let codetabs = tab.parentNode.parentNode;
   				  
   				  codetabs.querySelectorAll('.codetabs .tab.active').forEach(e => e.classList.remove('active'));
   				  codetabs.querySelectorAll('.codetabs .tab[data-codetab="' + tabId + '"]').forEach(e => e.classList.add('active'));  
   			 });
   		 });
   	 }
   </script>
   ```

## 使用说明

### 默认方式

在原始的`md`文件中添加`{{%/* codetabs */%}}  `和`{{%/* /codetabs */%}}`,之后按照常规的方式添加一个或多个代码块

```go { data-line="2,10,16" }
{{%/* codetabs */%}}  
​```java
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!"); 
    }
}
​```

​```php
<?php
echo "My first PHP script!";
?>
​```

​```swift
let s: String = "sample";
​```

{{%/* /codetabs */%}}
```

展示效果如下，可看出代码块已经被正确分组。

{{% codetabs %}}   

```java
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!"); 
    }
}
```


```php
<?php
echo "My first PHP script!";
?>
```

```swift
let s: String = "sample";
```

{{% /codetabs %}}

### 自定义tab头

前述方式虽然能正常生效，但tab表头是对应的语言名称，当需要对多个相同的语言进行分组时，显示的tab名称会重复，影响阅读与使用体验。
{{% codetabs %}}   

```java
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!"); 
    }
}
```


```php
<?php
echo "My first PHP script!";
?>
```

```java
Map<String, Map<String, Map<String, List<String>>>> map = list.stream()
            .collect(Collectors.groupingBy(CompanyEntity::getName,
                    Collectors.groupingBy(CompanyEntity::getLocationName,
                    Collectors.groupingBy(CompanyEntity::getOfficeName,
                            Collectors.mapping(CompanyEntity::getBuildingName, Collectors.toCollection(ArrayList::new))))));
```

{{% /codetabs %}}

此时可通过分隔符的方式设置自定义的tab名称，默认的分隔符为 `::`

```go { data-line="2,10,16" }
{{%/* codetabs */%}}  
​```java
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!"); 
    }
}
​```

​```php::世界上最好的语言
<?php
echo "My first PHP script!";
?>
​```

​```java::Lambda表达式分组
Map<String, Map<String, Map<String, List<String>>>> map = list.stream()
            .collect(Collectors.groupingBy(CompanyEntity::getName,
                    Collectors.groupingBy(CompanyEntity::getLocationName,
                    Collectors.groupingBy(CompanyEntity::getOfficeName,
                            Collectors.mapping(CompanyEntity::getBuildingName, Collectors.toCollection(ArrayList::new))))));
​```

{{%/* /codetabs */%}}
```

显示效果如下，可看出tab表头已经变的更有含义，同时也可根据需要只对部分代码块添加自定义表头


{{% codetabs %}}   

```java
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!"); 
    }
}
```


```php::世界上最好的语言
<?php
echo "My first PHP script!";
?>
```

```java::Lambda表达式分组
Map<String, Map<String, Map<String, List<String>>>> map = list.stream()
            .collect(Collectors.groupingBy(CompanyEntity::getName,
                    Collectors.groupingBy(CompanyEntity::getLocationName,
                    Collectors.groupingBy(CompanyEntity::getOfficeName,
                            Collectors.mapping(CompanyEntity::getBuildingName, Collectors.toCollection(ArrayList::new))))));
```

{{% /codetabs %}}

### tab分隔符设置

默认的tab名称分隔符为 `::`，系统还支持页面分隔符和全局分隔符，它们的优先级如下

`页面分隔符`>`全局分隔符`>`默认分隔符`

全局分隔符可通过在`config.toml`添加如下配置来实现

```toml
[params]
  codeTabSeperator = "#"
```

页面分隔符可通过在文章头部添加如下配置来实现

```properties
codeTabSeperator: "::"
```

