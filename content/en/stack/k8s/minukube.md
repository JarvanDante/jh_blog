---
author: "Karson"
title: "minikube安装(单机集群)"
date: 2022-05-21
description: "用于在本地计算机上运行单节点Kubernetes集群的工具"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags:
- docker 
categories:
- k8s
---


## 资源调整
原docker desktop中的配置：
1. 关闭k8x：勾掉 `Enable Kubernetes`
也可调配置：降低 2h4G

## 安装minikube
### 安装kubectl
地址：
`https://kubernetes.io/docs/tasks/tools/`
本是mac：
`https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/`
```shell
brew install kubectl
kubectl version --client

```
### 安装minikube
地址：
`https://minikube.sigs.k8s.io/docs/start/`
本是mac：
```shell
brew install minikube
#如果哪一个minikube在通过brew安装后失败，您可能需要删除旧的minikube链接并链接新安装的二进制文件：
brew unlink minikube
brew link minikube
```
### 启动minikube(开VPN)
```shell
minikube start --image-mirror-country='cn'
# 指定资源配置
$ minikube start --cpus=4 --memory=6000mb --image-mirror-country='cn'
😄  Darwin 14.0 (arm64) 上的 minikube v1.33.0
✨  根据现有的配置文件使用 docker 驱动程序
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.43 ...
🏃  正在更新运行中的 docker "minikube" container ...
    > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubeadm.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubelet.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubectl:  47.63 MiB / 47.63 MiB [-------------] 100.00% 5.03 MiB p/s 9.7s
    > kubeadm:  46.69 MiB / 46.69 MiB [------------] 100.00% 950.05 KiB p/s 51s
    > kubelet:  91.98 MiB / 91.98 MiB [----------] 100.00% 968.87 KiB p/s 1m37s

    ▪ 正在生成证书和密钥...
    ▪ 正在启动控制平面...
    ▪ 配置 RBAC 规则 ...
🔗  配置 bridge CNI (Container Networking Interface) ...
🔎  正在验证 Kubernetes 组件...
    ▪ 正在使用镜像 registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner:v5
🌟  启用插件： storage-provisioner, default-storageclass
🏄  完成！kubectl 现在已配置，默认使用"minikube"集群和"default"命名空间
```
minikube 提供了非常多的配置参数，
常用配置参数如下
+ --driver=*** 从1.5.0版本开始，Minikube缺省使用系统优选的驱动来创建Kubernetes本地环境，比如您已经安装过Docker环境，minikube 将使用 docker 驱动
+ --cpus=2: 为minikube虚拟机分配CPU核数
+ --memory=2048mb: 为minikube虚拟机分配内存数
+ --registry-mirror=*** 为了提升拉取Docker Hub镜像的稳定性，可以为 Docker daemon 配置镜像加速，参考阿里云镜像服务
+ --kubernetes-version=***: minikube 虚拟机将使用的 kubernetes 版本

### 验证minikube 
```shell
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

$ kubectl version --client
Client Version: v1.29.1
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3

# kubectl 的配置已经指向 minikube
$ kubectl config current-context 
minikube
# 删除minikube集群
kubectl config get-contexts
kubectl config use-context docker-desktop 切换其它群集


# kubectl 集群信息
kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:61296
CoreDNS is running at https://127.0.0.1:61296/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get no
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   7m51s   v1.30.0
```
### 界面dashboard
```shell
$ minikube dashboard
```
![/images/docImages/mi1.png](/images/docImages/mi1.png)
### 查看扩展列表
```shell
# 查看扩展列表
$ minikube addons list
|-----------------------------|----------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE  |    STATUS    |           MAINTAINER           |
|-----------------------------|----------|--------------|--------------------------------|
| ambassador                  | minikube | disabled     | 3rd party (Ambassador)         |
| auto-pause                  | minikube | disabled     | minikube                       |
| cloud-spanner               | minikube | disabled     | Google                         |
| csi-hostpath-driver         | minikube | disabled     | Kubernetes                     |
| dashboard                   | minikube | enabled ✅   | Kubernetes                     |
| default-storageclass        | minikube | enabled ✅   | Kubernetes                     |
```