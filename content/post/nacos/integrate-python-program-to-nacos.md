---
title: "在Python程序中使用Nacos"
date: 2023-04-11T09:25:53+08:00
lastmod: 2023-04-11T09:25:53+08:00
draft: false
keywords: ["nacos","python","flask"]
description: "简要介绍如何基于nacos官方的sdk将python程序集成到nacos中去"
tags: ["nacos","python"]
categories: ["python编程"]
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

简要介绍如何在基于[**Flask**](https://flask.palletsprojects.com/en/2.3.x/)开发的程序中通过[**Nacos**](https://nacos.io/en-us/)提供的[**nacos-sdk-python**](https://github.com/nacos-group/nacos-sdk-python)实现在`Python`程序中整合`Nacos`。

<!--more-->

完整代码在[**python-nacos-demo**](https://github.com/fox-world/python-nacos-demo)

## 注册Nacos

官网文档的说明如下

![Nacos注册实例](/blog_img/nacos/integrate-python-program-to-nacos/nacos-register-instance.png "Nacos注册实例") 

从中可看出只有`service_name`、`ip`与`port` 是必填项，而我们在实际使用中经常需要通过group进行区分，上述文档中并没有相关说明，翻看其源码发现`add_naming_instance`的方法签名如下，其中包含group相关信息，**文档没有与源码保持一致，必须吐槽下`Nacos`官方！**

```python
def add_naming_instance(self, service_name, ip, port, cluster_name=None, weight=1.0, metadata=None,
                        enable=True, healthy=True, ephemeral=True,group_name=DEFAULT_GROUP_NAME):
```



同时可发现要注册实例，必须先获得一个`NacosClient`对象之后才能进行对应操作，基于官方文档的实现如下：

```python
client = nacos.NacosClient(server_address, namespace=namespace)
client.add_naming_instance(service_name, service_address, port, group_name=group_name)
```

由于对`Nacos`的各种操作都涉及到`NacosClient`，故可将其定义为一个全局变量，后续在其它地方可直接使用

```python
global NACOS_CLIENT
NACOS_CLIENT = nacos.NacosClient(server_address, namespace=namespace)
NACOS_CLIENT.add_naming_instance(service_name, service_address, port, group_name=group_name)
```

### 程序启动后监听

安装完成后可在`Nacos`页面的服务菜单中看见对应的实例，但经过十几秒后该实例会显示为如下图所示的不健康状态，之后会自动从`Nacos`服务列表中移除掉：

![Nacos实例不健康](/blog_img/nacos/integrate-python-program-to-nacos/nacos-instance-not-health.png "Nacos实例不健康") 

造成此现象的原因为`nacos-sdk-python`没有提供自动发送心跳的机制，需要自己实现代码来持续不断地发送心跳请求以维持健康状态，可通过异步线程的方式实现，相关代码如下：

```python
NACOS_SERVER = None
NACOS_SERVICE = None
NACOS_CLIENT = None

logger = log.get_logger(__name__)


def register_nacos(yml_data):
    # 服务器配置
    port = yml_data['nacos']['server_port']
    server_address = yml_data['nacos']['server_address']
    global NACOS_SERVER
    NACOS_SERVER = NacosServer(port, server_address)

    # 调用方配置
    namespace = yml_data['nacos']['namespace']
    service_address = yml_data['nacos']['service_address']
    service_port = yml_data['nacos']['service_port']
    service_name = yml_data['nacos']['service_name']
    group_name = yml_data['nacos']['group_name']

    global NACOS_SERVICE
    NACOS_SERVICE = NacosService(namespace, group_name, service_name, service_address, service_port)

    global NACOS_CLIENT
    NACOS_CLIENT = nacos.NacosClient(server_address, namespace=namespace)
    NACOS_CLIENT.add_naming_instance(service_name, service_address, service_port, group_name=group_name)
    logger.info("=========register nacos success===========")

    thread = threading.Thread(target=send_heartbeat, name="send_heartbeat_threads",
                              args=(NACOS_CLIENT, service_name, service_address, service_port, group_name),
                              daemon=True)
    thread.start()


def send_heartbeat(client, service_name, ip, port, group_name):
    while True:
        client.send_heartbeat(service_name, ip, port, group_name=group_name)
        time.sleep(5)
```

## 读取配置

在`Java`中通过引入`spring-cloud-starter-alibaba-nacos-discovery`依赖可直接解析相关的配置文件，非常方便，而在`Python`中相对没这么简洁，其使用方法如下

![Nacos读取配置](/blog_img/nacos/integrate-python-program-to-nacos/nacos-get-config.png "Nacos读取配置") 

使用代码类似如下

```python
config = NACOS_CLIENT.get_config(data_id, group, no_snapshot=True)
```

其返回值是一个[**dictionary**](https://docs.python.org/3/tutorial/datastructures.html#dictionaries)对象，获取之后可根据实际情况做进一步处理

## 日志系统启用

在[**How do I obtain logs from a Nacos client**](https://www.alibabacloud.com/help/en/mse/support/how-do-i-obtain-logs-from-a-nacos-client#p-vhz-cfm-2xy)中有如下信息说明`Nacos`中的日志与[**logging**](https://docs.python.org/3/library/logging.html)模块保持一致，只需要通过`logging`开启日志即可。

> A Python Nacos client uses the logging module of Python, which is consistent with the logging module of your application. Logs of a Python Nacos client are displayed in application logs.

可在`app.py`程序的头部添加类似如下代码即可开启`Nacos`日志

```python
logger = logging.getLogger(name)
logger.setLevel(logging.INFO)
```

开启后显示效果类似如下

![Nacos日志输出](/blog_img/nacos/integrate-python-program-to-nacos/nacos-log-output.png "Nacos日志输出") 

## 调用其它程序

在`Spring`中可通过[**OpenFeign**](https://spring.io/projects/spring-cloud-openfeign)或[**Dubbo**](https://dubbo.apache.org/en/index.html)来调用注册到`Nacos`中的服务接口，由于`Python`与`Java`是两套不同的体系，在`Python`只能通过[**Requests**](https://pypi.org/project/requests/)模块直接发送对应的`HTTP`请求实现，类似如下代码：

```python
url = 'http://127.0.0.1:8081/user/queryAll'
response = requests.get(url).text
print(response)
```

在上述代码中，url地址中的`ip`和`port`以硬编码的形式指定，实际使用环境的`ip`和`port`会动态变化，显然此种方式使用起来不灵活。结合`Nacos`本身的特性，可先通过服务名称获取对应的服务实例信息，则其中解析出`ip`和`port`，之后发送请求，从而消除动态配置。

### 获取实例

获取实例的方法说明如下

![Nacos查询实例](/blog_img/nacos/integrate-python-program-to-nacos/nacos-query-instance.png "查询实例") 

相关调用代码类似如下:

```python
namespace = NACOS_SERVICE.namespace
group_name = NACOS_SERVICE.group_name
instances = NACOS_CLIENT.list_naming_instance(service_name, namespace_id=namespace,group_name=group_name,healthy_only=True)
```

执行后的结果类似如下，可通过在`hosts`中找到对应的`ip`和`port`为调用做准备

![Nacos查询实例结果](/blog_img/nacos/integrate-python-program-to-nacos/nacos-query-instance-result.png "Nacos查询实例结果") 

### 调用接口

获取到`ip`和`port`之后，接下来就是正常的发送`HTTP`请求，此时已经和`Nacos`没有关系

```python
instances = get_instance(service)
ip = instances['hosts'][0]['ip']
port = instances['hosts'][0]['port']
url = "http://" + ip + ":" + str(port) + "/" + method
response = requests.get(url).text
print(response)
```