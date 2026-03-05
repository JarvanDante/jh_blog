---
author: "jarvan"
title: "启动Kubernetes下载镜像失败一直starting"
date: 2022-05-21
description: "启动 Kubernetes 所需的镜像往往会下载失败，于是点击 Apply 后，该配置页面的右下角始终显示 Kubernetes is starting，无法正常启动。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags:
- k8s 
categories:

---


## starting失败原因
`Kubernetes is starting`

## 解决方案
### 拉取仓库k8s-docker-desktop-for-mac
我们先将该仓库拉取到本地：
```shell
git clone git@github.com:maguowei/k8s-docker-desktop-for-mac.git
```

### 查看版本
然后确认一下 Docker Desktop 自带的 Kubernetes 的版本。点击 Docker 图标，选择 About Docker Desktop，看到如下界面：
![/images/docImages/k8s1.png](/images/docImages/k8s1.png)

可以看到 Kubernetes 的版本是 Kubernetes: v1.29.1。

之后我们打开 k8s-docker-desktop-for-mac 项目下的 images 文件：
```shell
$cat images
k8s.gcr.io/kube-proxy:v1.25.2=gotok8s/kube-proxy:v1.25.2
k8s.gcr.io/kube-controller-manager:v1.25.2=gotok8s/kube-controller-manager:v1.25.2
k8s.gcr.io/kube-scheduler:v1.25.2=gotok8s/kube-scheduler:v1.25.2
k8s.gcr.io/kube-apiserver:v1.25.2=gotok8s/kube-apiserver:v1.25.2
k8s.gcr.io/coredns:v1.9.3=gotok8s/coredns:v1.9.3
k8s.gcr.io/pause:3.8=gotok8s/pause:3.8
k8s.gcr.io/etcd:3.5.4-0=gotok8s/etcd:3.5.4-0%
```

### 获取k8s版本镜像
确保文件中的 Kubernetes 版本号与 Docker Desktop 自带的 Kubernetes 版本号一致后，执行命令：
```shell
./load_images.sh
```
该命令会帮助我们拉取启动 Kubernetes 所需的所有镜像。
命令执行完毕后，点击 Docker 图标，在 Preferences.. > Reset 界面中点击 Reset Kubernetes cluster，重启 Kubernetes。大功告成！

### 验证成功
```shell
$kubectl cluster-info
Kubernetes control plane is running at https://kubernetes.docker.internal:6443
CoreDNS is running at https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```