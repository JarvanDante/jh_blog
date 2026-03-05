---
author: "Karson"
title: "consul安装"
date: 2022-12-03 00:00:01
description: "Consul 简化了分布式环境中的服务的注册和发现流程，通过 HTTP 或者 DNS 接口发现"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags: 
- golang
categories:
- consul




---

## consul
官方网站：
https://consul.io/

### 安装consul
1. 方式一：macOS系统：
```shell
brew tap hashicorp/tap
brew install hashicorp/tap/consul
//检查版本号
consul version
```
2. 方式二：linux系统：

+ Debian--
```shell
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/knowledge/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install consul
```
+ CentOS--
```shell
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install consul
```
+ Fedora--
```shell
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo dnf -y install consul
```

### 开启consul
#### 启动Consul Server
命令：
```shell
# node1
consul agent -server -bootstrap-expect 2 -data-dir /tmp/consul -node=n1 -bind=192.168.110.123 -ui -config-dir /etc/consul.d -rejoin -join 192.168.110.123 -client 0.0.0.0
```
说明
#运行consul agent以server模式
+ -server : 定义agent运行在server模式
+ -bootstrap-expect : 在一个datacenter中期望提供的server节点数目，当该值提供的时候，consul一直等到达到指定server数目的时候才会引导整个集群，该标记不能和bootstrap共用
+ -data-dir : 提供一个目录用来存放agent的状态，所有的agent允许都需要该目录，该目录必须是稳定的，系统重启后都继续存在
+ -node : 节点在集群中的名称，在一个集群中必须是唯一的，默认是该节点的主机名
+ -bind : 该地址用来在集群内部的通讯，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0
+ -ui : 启动web界面
+ -config-dir : 配置文件目录，里面所有以.json结尾的文件都会被加载
+ -rejoin : 使consul忽略先前的离开，在再次启动后仍旧尝试加入集群中
+ -client : consul服务侦听地址，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1所以不对外提供服务，如果你要对外提供服务改成0.0.0.0

```shell
# node2
consul agent -server -bootstrap-expect 2 -data-dir /tmp/consul -node=n2 -bind=192.168.110.156 -ui -rejoin -join 192.168.110.123
```

#### 启动Consul Client

运行consul agent以clent模式，-join 加入到已有的集群中去
```shell
# node3
consul agent -data-dir /tmp/consul -node=n3 -bind=192.168.110.124 -config-dir /etc/consul.d -rejoin -join 192.168.110.123
```

#### 验证启动
```shell
//启动consul
consul agent -dev

...
2023-11-17T16:25:34.985+0800 [INFO]  agent: Starting server: address=127.0.0.1:8500 network=tcp protocol=http
2023-11-17T16:25:35.025+0800 [INFO]  agent: Started gRPC listeners: port_name=grpc_tls address=127.0.0.1:8503 network=tcp
2023-11-17T16:25:35.025+0800 [INFO]  agent: Started gRPC listeners: port_name=grpc address=127.0.0.1:8502 network=tcp
```
访问浏览器：
http://127.0.0.1:8500/

查看命令：

<font color="lightgreen">consul members</font> -- 查看节点
```shell
consul members
Node                     Address         Status  Type    Build   Protocol  DC   Partition  Segment
wangdeMacBook-Pro.local  127.0.0.1:8301  alive   server  1.17.0  2         dc1  default    <all>
```
<font color="lightgreen">consul leave</font> -- 优雅退出
如果一个agent作为一个服务器，一个优雅的离开是很重要的。避免引起潜在的可用性故障影响达成一致性协议。

<font color="lightgreen">dig @127.0.0.1 -p 8600 web.service.consul SRV</font> -- 查看主机信息
```shell
$ dig @127.0.0.1 -p 8600 web.service.consul SRV
; <<>> DiG 9.10.6 <<>> @127.0.0.1 -p 8600 web.service.consul SRV
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 2157
;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;web.service.consul.            IN      SRV

;; AUTHORITY SECTION:
consul.                 0       IN      SOA     ns.consul. hostmaster.consul. 1700214831 3600 600 86400 0

;; Query time: 0 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Fri Nov 17 17:53:51 CST 2023
;; MSG SIZE  rcvd: 97
```