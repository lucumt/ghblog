---
title: "利用Nacos与KubeSphere创建多套开发与测试环境"
date: 2022-09-15T15:46:24+08:00
lastmod: 2022-09-15T15:46:24+08:00
draft: false
keywords: ["kubesphere","jenkins","nacos","动态配置","多套环境"]
description: "利用Nacos与KubeSphere创建多套开发与测试环境"
tags: ["kubesphere","jenkins","nacos"]
categories: ["持续集成"]
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

---

基于[KubeSphere使用心得](/post/devops/share-experiences-for-using-kubesphere/)给部门搭建了`dev`、`sit`、`test`、`prod`这4套环境之后，一开始使用较为顺利，但随着项目的推进以及开发人员的增多，同时有多个功能模块需要并行开发与测试，导致原有的4套环境不够用。经过一番摸索后，实现了结合[Nacos](https://nacos.io/zh-cn/docs/what-is-nacos.html)在`KubeSphere`中动态配置多套环境功能，通过修改`Nacos`中的`JSON`配置文件可很容易的从4套扩展为16套甚至更多。

<!--more-->

## 原实现方式

最开始自己只准备了`dev`、`sit`、`test`、`prod`这4套环境，由于环境数量不多，对于不同环境的端口配置自己是在代码中直接实现的[^1]

```groovy
 switch(PRODUCT_PHASE) {
     case "dev":
     env.NODE_PORT = 12002
     break
     case "sit":
     env.NODE_PORT = 13002
     break
     case "test":
     env.NODE_PORT = 14002
     break
     case "prod":
     env.NODE_PORT = 15002
     break
 }
```

此种方式将相关环境相关的配置全都集中到`Jenkins`流水线中，在最初的使用阶段可减少配置文件数量，能够快速编写流水线，快速交付使用。但随着项目规模与人员的扩大，当需要灵活配置多套环境时，此种方式采用硬编码的方式会显得捉襟见肘。

![kubesphere原始部署方式](/blog_img/devops/using-nacos-and-kubesphere-to-create-multiple-environments/kubesphere-build-without-nacos-config.png "kubesphere原始部署方式") 

## 修改后的方式

结合项目实际情况以及避免后续再次修改`KubeSphere`流水线，为了实现**灵活的配置多套环境**，自己制定了如下2个规则：

1. 端口信息存放到配置文件中，`KubeSphere`在构建时去流水线读取相关配置
2. 当需要扩展环境或修改端口时，不需要修改`KubeSphere`中的流水线，只需要修改对应的端口配置文件即可[^2]

由于项目中采用`Nacos`作为配置中心与服务管理平台，故决定采用`Nacos`作为端口的配置中心，实现流程如下：

![kubesphere结合nacos部署](/blog_img/devops/using-nacos-and-kubesphere-to-create-multiple-environments/kubesphere-build-with-nacos-config.png "kubesphere结合nacos部署") 

基于上述流程，在个人项目中面临如下问题：

* 利用`Groovy`代码获取`Nacos`中特定的端口`JSON`配置文件，并能动态解析
* 利用`Groovy`代码根据输入输入参数动态的获取`Nacos`中对应的`namespace`
* 由于环境的增多，不可能每套环境都准备一个`YAML`文件，此时需要动态的读取并更新`YAML`文件

### 安装Pipeline Utility Steps插件

`Jenkins`默认不支持`JSON`、`YAML`的解析，需要在`Jenkins`中预先安装[Pipeline Utility Steps](https://www.jenkins.io/doc/pipeline/steps/pipeline-utility-steps/)插件，该插件提供了对`JSON`、`YAML`、`CSV`、`PROPERTIES`等常见文件格式的读取与修改操作。

### JSON文件设计

`JSON`文件设计如下，通过env、server、dubbo等属性记录环境和端口信息，通过project来记录具体的项目名称，由于配置文件中的key都是固定的，后续`Groovy`解析时会较为方便，在需要扩展环境时只需要更新此`JSON`文件即可。

```json
{
    "portConfig":[
        {
            "project":"lucumt-system",
            "ports":[
                {
                    "env":"dev-1",
                    "server":12001,
                    "dubbo":12002
                },
                {
                    "env":"dev-2",
                    "server":12201,
                    "dubbo":12202
                }
            ]
        },
        {
            "project":"lucumt-idp",
            "ports":[
                {
                    "env":"dev-1",
                    "server":13001,
                    "dubbo":13002
                },
                {
                    "env":"dev-2",
                    "server":13201,
                    "dubbo":13202
                }
            ]
        }
    ]
}
```

### 读取namespace

在[Nacos Open Api](https://nacos.io/zh-cn/docs/open-api.html)中可知查询`namespace`的请求为`/nacos/v1/console/namespaces`，基于`Groovy`的读取代码如下：

```groovy
response = sh(script: "curl -X GET 'http://xxx.xxx.xxx.xxx:8848/nacos/v1/console/namespaces'", returnStdout: true)
jsonData = readJSON text: response
namespaces = jsonData.data
for(nm in namespaces){
    if(BUILD_TYPE==nm.namespaceShowName){
        NACOS_NAMESPACE = nm.namespace
    }
}
```

### 读取Nacos配置文件

在[Nacos Open Api](https://nacos.io/zh-cn/docs/open-api.html)中可知查询配置文件的请求为`/nacos/v1/cs/configs`，基于`Groovy`的读取代码如下：

```groovy
response = sh(script: "curl -X GET 'http://xxx.xxx.xxx.xxx:8848/nacos/v1/cs/configs?dataId=idp-custom-config.json&group=idp-custom-config&tenant=0f894ca6-4231-43dd-b9f3-960c02ad20fa'", returnStdout: true)
jsonData = readJSON text: response
configs = jsonData.portConfig
for(config in configs){
    project = config.project
    if(project!=PROJECT_NAME){
       continue
    }
    ports = config.ports
    for(port in ports){
        if(port.env!=BUILD_TYPE){
            continue
        }
        env.NODE_PORT = port.server
	}
}
```

### 根据YAML文件

由于自己将项目中变化的部分已经单独抽取为了一个`YAML`文件，故只需要修改此单独配置文件即可，在修改过程中，处于简化考虑，自己先将原有的`YAML`文件删除，之后重新写入，相关代码如下

```groovy
yamlFile = 'src/main/resources/bootstrap-dev.yml'
yamlData = readYaml file: yamlFile
yamlData.spring.cloud.nacos.discovery.group = BUILD_TYPE
yamlData.spring.cloud.nacos.discovery.namespace = NACOS_NAMESPACE
yamlData.spring.cloud.nacos.config.namespace = NACOS_NAMESPACE
sh "rm $yamlFile"

writeYaml file: yamlFile, data: yamlData
```

### 运行效果

上述的配置均需要在项目编译之前进行，配置完毕的流水线运行效果如下，程序可正常运行，改造目的实现。

![kubesphere多环境运行](/blog_img/devops/using-nacos-and-kubesphere-to-create-multiple-environments/kubesphere-build-with-multiplep-environments.png "kubesphere多环境运行") 

## 参考代码

* **前端Jenkins流水线**，参见[lucumt-system-web-new.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/lucumt-system-web-new.groovy)
* **后端Jenkins流水线**，参见[lucumt-system-new.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/lucumt-system-new.groovy)

[^1]: 参见[lucumt-system.groovy](https://github.com/lucumt/myrepository/blob/master/jenkins/lucumt-system.groovy)
[^2]:  在个人项目中`Jenkins`流水线不需要修改，但需要修改`Kubesphere`中的默认输入配置，将新增的环境加到下拉列表中，便于使用