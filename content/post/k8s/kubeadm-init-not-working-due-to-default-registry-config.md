---
title: "利用kubeadm init初始化时由于registry.k8s.io/pause:3.6导致初始化失败"
date: 2023-01-11T11:31:45+08:00
lastmod: 2023-01-11T11:31:45+08:00
draft: false
keywords: ["kubernetes","linux","kubeadm init","registry.k8s.io"]
description: "记录利用kubeadm init初始化时由于registry.k8s.io/pause:3.6导致初始化失败的解决过程"
tags: ["kubernetes","linux"]
categories: ["容器化"]
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

---

工作中涉及到`Kubernetes`相关知识，自己之前一直没有系统性的学习`Kubernetes`，近期在腾讯云上想安装`Kubernetes`时一直遇到在执行`kubeadm init`时`6443`和`10280`端口无法访问导致操作失败进而无法顺利安装`Kubernetes`。一番排查后发现是由于从`1.24.0`之后`Kubernetes`默认采用`containerd`作为运行时容器，其默认镜像为`registry.k8s.io`，该镜像在国内无法访问导致的，简单记录下。

<!--more-->

安装过程主要参考[^1]，相关配置与安装步骤如下：

## 系统配置与依赖

### 关闭防火墙

```bash
# 关闭交换内存
swapoff -a

#关闭selinux
getenforce
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config

#关闭防火墙
firewall-cmd --state 
systemctl stop firewalld.service 
systemctl disable firewalld.service 
```

### 配置docker参数

```bash
mkdir /etc/docker/
vim /etc/docker/daemon.json

{
  "storage-driver": "overlay2",
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

### 配置内核参数

```bash
## 配置网卡转发,看值是否为1
sysctl -a |grep 'net.ipv4.ip_forward = 1'
sysctl -a |grep 'net.bridge.bridge-nf-call-iptables = 1'
sysctl -a |grep 'net.bridge.bridge-nf-call-ip6tables = 1'

## 若未配置，需要执行如下
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1
EOF

sysctl -p /etc/sysctl.d/k8s.conf
```

### 安装docker

```bash
#安装相关依赖
yum install -y yum-utils device-mapper-persistent-data lvm2 epel-release

#添加阿里云docker-ce源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum clean all
yum install -y docker-ce containerd.io

#设置docker开机自启
systemctl enable docker

#启动docker服务
systemctl start docker

#查看docker信息
docker info
```

### 配置container

```bash
containerd config default > /etc/containerd/config.toml
systemctl daemon-reload
systemctl restart containerd
```

## 安装Kubernetes

### 添加k8s源

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 安装kubelet和kubeadm

1. 执行下述指令，安装`kubelet kubeadm`

   ```bash
   yum -y install kubectl kubelet kubeadm
   ```

2. 执行下述指令，修改`kubelet`配置 **（把kubelet驱动方式改为和docker驱动方式一致，否则会有报错）**

   ```bash
   cat <<EOF >/etc/sysconfig/kubelet
   KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
   EOF
   ```

3. 执行下述执行添加自启动

   ```bash
   systemctl enable kubelet
   ```

4. 前序步骤执行完成后，执行`kubectl version`输出结果如下，可以看出其无法访问`8080`端口，结果输出不完整。造成此现象的原因为没有执行`kubeadm init`

   ![kubectl version输出结果不完善](/blog_img/k8s/kubeadm-init-not-working-due-to-default-registry-config/kubectl-version-output-1.png "kubectl version输出结果不完善")  

### 执行kubeadm init

1. 执行下述命令进行初始化

   ```bash
   kubeadm init --v=5  --upload-certs --image-repository k8s.m.daocloud.io
   ```

2. 前述命令的输出如下，可以看出创建过程一直阻塞

   ![kubeadm init创建过程阻塞](/blog_img/k8s/kubeadm-init-not-working-due-to-default-registry-config/kubeadm-init-waiting-to-boot-up-control-plane.png "kubeadm init创建过程阻塞")   

3. 经过一段时间的等待，最终在控制台中出现如下错误信息，`kubeadm init`过程失败!

   ![kubeadm init创建失败](/blog_img/k8s/kubeadm-init-not-working-due-to-default-registry-config/kubeadm-init-failed-output-log.png "kubeadm init创建失败")  

## 问题分析&解决

1. 由于原始的报错信息没有提供太多有用的信息，故通过下述命令来输出其完整的日志[^2]

   ```bash
   systemctl status kubelet -l > logs.txt
   cat logs.txt
   ```

2. 查看logs.txt的内容，输出如下

   ![kubeadm init找不到镜像导致部署出错](/blog_img/k8s/kubeadm-init-not-working-due-to-default-registry-config/kubeadm-init-failed-to-plull-image.png "kubeadm init找不到镜像导致部署出错")

3. 从上图中可知报错信息是由于无法下载`registry.k8s.io/pause:3.6`导致的，由于`registry.k8s.io`在国内被墙，但是自己在执行`kubeadm init`的时候已经通过`--image-repository k8s.m.daocloud.io`制定了国内的镜像，为啥还会访问旧的镜像呢？奇怪！

4. 网上搜索一番后，在其官网说明中[^3]的首页有如下说明

   > After its deprecation in v1.20, the dockershim component has been removed from the kubelet. From v1.24 onwards, you will need to either use one of the other [supported runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) (such as containerd or CRI-O) or use cri-dockerd if you are relying on Docker Engine as your container runtime. For more information about ensuring your cluster is ready for this removal, please see [this guide](https://kubernetes.io/blog/2022/03/31/ready-for-dockershim-removal/).

   而在前面通过`kubectl version`命令可知当前的版本为`v1.26.0`，很明显其使用的是`containerd`，而自己没有对其进行相关配置。

5. 执行`cat /etc/containerd/config.toml|grep registry.k8s.io`后的结果如下，`containerd`使用的还是默认镜像，至此问题原因找出！

   ![ontainerd使用的是默认镜像](/blog_img/k8s/kubeadm-init-not-working-due-to-default-registry-config/containerd-config-output.png "containerd使用的是默认镜像")     

6. 依次执行下述命令来重新配置`containerd`

   ```bash
   sed -i 's/registry.k8s.io/registry.aliyuncs.com\/google_containers/' /etc/containerd/config.toml
   systemctl daemon-reload
   systemctl restart containerd
   ```

7. 之后重新执行下述命令

   ```bash
   # 移除之前的旧配置
   kubeadm reset -f
   
   # 初始化
   kubeadm init --v=5  --upload-certs --image-repository k8s.m.daocloud.io
   ```

8. 运行上述命令后的输出类似如下，可以看出`kubeadm init`顺利执行成功，整个过程耗时不超过1分钟，至此问题解决!

   ![kubeadm init执行成功](/blog_img/k8s/kubeadm-init-not-working-due-to-default-registry-config/kubeadm-init-execute-success.png "kubeadm init执行成功")   

[^1]: https://www.cnblogs.com/cerberus43/p/15881294.html
[^2]: 之所以要写入文件是由于直接执行`systemctl status kubelet`时其输出信息较多，在屏幕中显示不完整，不利于分析
[^3]: https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md