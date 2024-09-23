---
title: "在Docker容器内部获取当前容器实例的名称"
date: 2021-05-06T13:16:52+08:00
lastmod: 2021-05-06T13:16:52+08:00
draft: false
keywords: ["docker","容器名称","socket"]
description: "简要说明如何实现在Docker容器内部获取当前容器的名称,包含基于环境变量的动态实现以及基于Socket通信的动态实现"
tags: ["docker"]
categories: ["容器化"]
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

介绍如何在`Docker`容器内部获取当前容器名称的方法，特别感谢[**Get container name inside the docker container**](https://medium.com/disassembly/get-container-name-inside-the-docker-container-1997d36aaecd)。

<!--more-->

## 背景

某个项目模块采用了分布式部署，对于某个工程模块部署了多个副本，生成了多个`Docker`容器，在查看系统日志时，需要能够显示出具体是哪个容器执行的。

虽然采用`Docker`容器对应的64位或12位的字符串id也可唯一标识，但在阅读时不太直观，用容器名称会更直观。

如果在容器外，可通过`docker inspect --format='{{.Name}}' 容器id`指令很容易的获取容器名称，而在容器内部可通过`echo $HOSTNAME`快速的获取容器id，但对于容器名称的获取却不是很方便。

## 通过环境变量

直接通过`-e`命令将容器名称传入，然后通过`$`获取容器名称。

```bash
# 指定名称创建容器
docker run -e "containerName=nginx-custom-test" --name nginx-custom-test -d nginx

# 直接在宿主机中执行，不进入容器内部
docker exec nginx-custom-test env | grep containerName

# 先进入容器内部，然后执行
docker exec -it nginx-custom-test /bin/bash
echo $containerName
```

![通过环境变量获取指定的容器名](/blog_img/docker/get-docker-name-inside-docker-container/get-specified-container-name-via-env.png  "通过环境变量获取指定的容器名")

此种方式实现起来比较简单，但最大的问题是容器名称需要提前设置好，不太灵活，当容器名称随机生成时，此种方式就彻底失效了。

## 通过socket通信

```bash
# 容器名称随机生成
docker run -v /run/docker.sock:/run/docker.sock:ro -d nginx

# 进入容器内部
docker exec -it [容器id] /bin/bash

# 在容器里面执行下述命令

# 获取容器的详细信息
dockerInfo=$(curl -s --unix-socket /run/docker.sock http://docker/containers/$HOSTNAME/json)

# 利用awk获取name名称
containerName=$(echo $dockerInfo |  awk -F'[:,}]' '{for(i=1;i<=NF;i++){if($i=="\"Name\""){print $(i+1);exit} }}')

# 格式化输出
echo "This is container $containerName" | tr -d '"/'
```

执行结果如下，可看出对于随机生成的容器名也能正常获取。

![通过socket获取随机生成的容器名](/blog_img/docker/get-docker-name-inside-docker-container/get-random-container-name-via-sockets.png  "通过socket获取随机生成的容器名")

`$dockerInfo`的格式化输出如下

```json
{
    "Id": "c6b9dbccd51c211690194f13a8f37c7bc6858aa7c8d627bec11faa143d315bba",
    "Created": "2024-04-23T07:30:23.641193169Z",
    "Path": "/docker-entrypoint.sh",
    "Args": [
        "nginx",
        "-g",
        "daemon off;"
    ],
    "State": {
        "Status": "running",
        "Running": true,
        "Paused": false,
        "Restarting": false,
        "OOMKilled": false,
        "Dead": false,
        "Pid": 17102,
        "ExitCode": 0,
        "Error": "",
        "StartedAt": "2024-04-23T07:30:23.885312035Z",
        "FinishedAt": "0001-01-01T00:00:00Z"
    },
    "Image": "sha256:2ac752d7aeb1d9281f708e7c51501c41baf90de15ffc9bca7c5d38b8da41b580",
    "ResolvConfPath": "/var/lib/docker/containers/c6b9dbccd51c211690194f13a8f37c7bc6858aa7c8d627bec11faa143d315bba/resolv.conf",
    "HostnamePath": "/var/lib/docker/containers/c6b9dbccd51c211690194f13a8f37c7bc6858aa7c8d627bec11faa143d315bba/hostname",
    "HostsPath": "/var/lib/docker/containers/c6b9dbccd51c211690194f13a8f37c7bc6858aa7c8d627bec11faa143d315bba/hosts",
    "LogPath": "/var/lib/docker/containers/c6b9dbccd51c211690194f13a8f37c7bc6858aa7c8d627bec11faa143d315bba/c6b9dbccd51c211690194f13a8f37c7bc6858aa7c8d627bec11faa143d315bba-json.log",
    "Name": "/exciting_hoover",
    "RestartCount": 0,
    "Driver": "overlay2",
    "Platform": "linux",
    "MountLabel": "",
    "ProcessLabel": "",
    "AppArmorProfile": "",
    "ExecIDs": [
        "95bf150c2a4f263e42df000693b51b00b583ebba7228044f6bd1bcdb86c1a22d"
    ],
    "HostConfig": {
        "Binds": [
            "/run/docker.sock:/run/docker.sock:ro"
        ],
        "ContainerIDFile": "",
        "LogConfig": {
            "Type": "json-file",
            "Config": {

            }
        },
        "NetworkMode": "default",
        "PortBindings": {

        },
        "RestartPolicy": {
            "Name": "no",
            "MaximumRetryCount": 0
        },
        "AutoRemove": false,
        "VolumeDriver": "",
        "VolumesFrom": null,
        "ConsoleSize": [
            59,
            185
        ],
        "CapAdd": null,
        "CapDrop": null,
        "CgroupnsMode": "host",
        "Dns": [

        ],
        "DnsOptions": [

        ],
        "DnsSearch": [

        ],
        "ExtraHosts": null,
        "GroupAdd": null,
        "IpcMode": "private",
        "Cgroup": "",
        "Links": null,
        "OomScoreAdj": 0,
        "PidMode": "",
        "Privileged": false,
        "PublishAllPorts": false,
        "ReadonlyRootfs": false,
        "SecurityOpt": null,
        "UTSMode": "",
        "UsernsMode": "",
        "ShmSize": 67108864,
        "Runtime": "runc",
        "Isolation": "",
        "CpuShares": 0,
        "Memory": 0,
        "NanoCpus": 0,
        "CgroupParent": "",
        "BlkioWeight": 0,
        "BlkioWeightDevice": [

        ],
        "BlkioDeviceReadBps": [

        ],
        "BlkioDeviceWriteBps": [

        ],
        "BlkioDeviceReadIOps": [

        ],
        "BlkioDeviceWriteIOps": [

        ],
        "CpuPeriod": 0,
        "CpuQuota": 0,
        "CpuRealtimePeriod": 0,
        "CpuRealtimeRuntime": 0,
        "CpusetCpus": "",
        "CpusetMems": "",
        "Devices": [

        ],
        "DeviceCgroupRules": null,
        "DeviceRequests": null,
        "MemoryReservation": 0,
        "MemorySwap": 0,
        "MemorySwappiness": null,
        "OomKillDisable": false,
        "PidsLimit": null,
        "Ulimits": [

        ],
        "CpuCount": 0,
        "CpuPercent": 0,
        "IOMaximumIOps": 0,
        "IOMaximumBandwidth": 0,
        "MaskedPaths": [
            "/proc/asound",
            "/proc/acpi",
            "/proc/kcore",
            "/proc/keys",
            "/proc/latency_stats",
            "/proc/timer_list",
            "/proc/timer_stats",
            "/proc/sched_debug",
            "/proc/scsi",
            "/sys/firmware",
            "/sys/devices/virtual/powercap"
        ],
        "ReadonlyPaths": [
            "/proc/bus",
            "/proc/fs",
            "/proc/irq",
            "/proc/sys",
            "/proc/sysrq-trigger"
        ]
    },
    "GraphDriver": {
        "Data": {
            "LowerDir": "/var/lib/docker/overlay2/d2b523e9b47aed738b4015a9591b8f4e17065b9f779cdec6cebd3250e0576e56-init/diff:/var/lib/docker/overlay2/68ad234bdb5026df1c2dfc72f7d26cfeac037e4684461b3428996d76a351b89b/diff:/var/lib/docker/overlay2/ab73b553d5915120731a9c64a25f8fa51b6530e7e467403d122e362a231b766f/diff:/var/lib/docker/overlay2/07a24b0cb136ba5bed299dcc16155f25e43760610029743554577e524ec144b1/diff:/var/lib/docker/overlay2/034c1bbdaa560ad684ad6de966f49c4e9a0cb434717476446ab27ecad5e1a495/diff:/var/lib/docker/overlay2/c631330de17b59a6e7836e832a4dff4abe5068955f8bf6441dffd72bc1679111/diff:/var/lib/docker/overlay2/f937882465a97a6b70532e87a6ed43281e94bfbd66bdf8d5a79e9842ae7e12f8/diff:/var/lib/docker/overlay2/6ef77dde042f0b96eb12de6b5dc8c7224c91afbce6716cdcadd449ab7cb2b7d0/diff",
            "MergedDir": "/var/lib/docker/overlay2/d2b523e9b47aed738b4015a9591b8f4e17065b9f779cdec6cebd3250e0576e56/merged",
            "UpperDir": "/var/lib/docker/overlay2/d2b523e9b47aed738b4015a9591b8f4e17065b9f779cdec6cebd3250e0576e56/diff",
            "WorkDir": "/var/lib/docker/overlay2/d2b523e9b47aed738b4015a9591b8f4e17065b9f779cdec6cebd3250e0576e56/work"
        },
        "Name": "overlay2"
    },
    "Mounts": [
        {
            "Type": "bind",
            "Source": "/run/docker.sock",
            "Destination": "/run/docker.sock",
            "Mode": "ro",
            "RW": false,
            "Propagation": "rprivate"
        }
    ],
    "Config": {
        "Hostname": "c6b9dbccd51c",
        "Domainname": "",
        "User": "",
        "AttachStdin": false,
        "AttachStdout": false,
        "AttachStderr": false,
        "ExposedPorts": {
            "80/tcp": {

            }
        },
        "Tty": false,
        "OpenStdin": false,
        "StdinOnce": false,
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            "NGINX_VERSION=1.25.5",
            "NJS_VERSION=0.8.4",
            "PKG_RELEASE=1~bookworm"
        ],
        "Cmd": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "Image": "nginx",
        "Volumes": null,
        "WorkingDir": "",
        "Entrypoint": [
            "/docker-entrypoint.sh"
        ],
        "OnBuild": null,
        "Labels": {
            "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
        },
        "StopSignal": "SIGQUIT"
    },
    "NetworkSettings": {
        "Bridge": "",
        "SandboxID": "9961035f980f2ce740f148a6db9ca19bcfba040bb42e7eba8f36704df4f7f473",
        "SandboxKey": "/var/run/docker/netns/9961035f980f",
        "Ports": {
            "80/tcp": null
        },
        "HairpinMode": false,
        "LinkLocalIPv6Address": "",
        "LinkLocalIPv6PrefixLen": 0,
        "SecondaryIPAddresses": null,
        "SecondaryIPv6Addresses": null,
        "EndpointID": "87973a9191829be1647399a85ed29c615bf86dbb3f8952fce326ba404a93c473",
        "Gateway": "172.17.0.1",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "IPAddress": "172.17.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "MacAddress": "02:42:ac:11:00:02",
        "Networks": {
            "bridge": {
                "IPAMConfig": null,
                "Links": null,
                "Aliases": null,
                "MacAddress": "02:42:ac:11:00:02",
                "NetworkID": "41c4546cd26c02078696ba9e74b0ee9e737778f82bfce41c5ab557dd8666a533",
                "EndpointID": "87973a9191829be1647399a85ed29c615bf86dbb3f8952fce326ba404a93c473",
                "Gateway": "172.17.0.1",
                "IPAddress": "172.17.0.2",
                "IPPrefixLen": 16,
                "IPv6Gateway": "",
                "GlobalIPv6Address": "",
                "GlobalIPv6PrefixLen": 0,
                "DriverOpts": null,
                "DNSNames": null
            }
        }
    }
}
```

