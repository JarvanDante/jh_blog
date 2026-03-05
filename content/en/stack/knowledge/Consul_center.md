---
author: "Karson"
title: "consul服务注册中心"
date: 2023-05-19
description: "consul服务注册中心"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags:
- knowledge
categories:
- consul
---

## 介绍
市面上常用的嗠注册中心有：
+ Eureka
+ Nacos
+ Consul
+ ZooKeeper
+ Etcd
+ CoreDNS

>Eureka2.X版本官方已经停止更新了。
 
Consul是HashiCorp公司推出使用go语言开发的开源工具,，用于实现分布式系统的服务发现与配置。 Consul是 <font color='cyan'>**分布式的**</font> 、<font color='cyan'>**高可用的**</font> 、 <font color='cyan'>**可横向扩展的**</font>。

## consul特性 
+ <font color='cyan'>**Raft算法**</font>：
>PS:Raft实现了和Paxos相同的功能，它将一致性分解为多个子问题：Leader选举（Leader election）、日志同步（Log replication）、安全性（Safety）、日志压缩（Log compaction）、成员变更（Membership change）等
+ <font color='cyan'>**服务发现**</font>：
consul通过DNS或者HTTP接口使服务注册和服务发现变的很容易，一些外部服务，例如saas提供的也可以一样注册。
+ <font color='cyan'>**健康检查**</font> ：
健康检测使consul可以快速的告警在集群中的操作。和服务发现的集成，可以防止服务转发到故障的服务上面。
+ <font color='cyan'>**Key/Value存储**</font> ：
一个用来存储动态配置的系统。提供简单的HTTP接口，可以在任何地方操作。
+ <font color='cyan'>**多数据中心**</font> ：
无需复杂的配置，即可支持任意数量的区域。
+ <font color='cyan'>**支持http和dns协议接口**</font> 
+ <font color='cyan'>**官方提供web管理界面**</font> 

## consul角色
### client:客户端
<font color='cyan'>**无状态**</font>，将 <font color='cyan'>**HTTP和DNS接口**</font> 请求转发给局域网的服务端集群。

### server:服务端
<font color='cyan'>**保存配置信息**</font>，<font color='cyan'>**高可用集群**</font>，每个数据中心的server数量推荐为3个或5个

![/images/docImages/cl1.png](/images/docImages/cl1.png)

## consul工作原理
### 1.服务发现以及注册
当服务 Producer 启动时，会将自己的 IP/host 等信息通过发送请求告知 Consul 、 Consul 接收到 Producer 的注册信息后，每隔10s(默认)会向 Producer 发送一个健康检查的请求，检验 Producer 是否健康。

### 2.服务调用
当 Consumer 请求 Producer 时，会先从 Consul 中拿到存储 Producer 服务的 IP和Port 的临时表（temp table）,从temp table 表中任选一个 Producer 的IP和Port,然后根据这个IP和Port，发送访问请求；temp table表只包含通过了健康检查的 Producer 信息，并且每隔10s（默认）更新。


![/images/docImages/cl2.png](/images/docImages/cl2.png)

## consul安装(本地)
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

## consul安装(docker-compose)
`vim docker-compose.yml`
```yml
version: '3.5'
services:
  consul1:
    image: consul:latest
    container_name: consul1
    restart: always
    command: agent -server -client=0.0.0.0 -bootstrap-expect=3 -node=consul1
    volumes:
      - /usr/local/docker_my/consul/consul1/data:/consul/data
      - /usr/local/docker_my/consul/consul1/config:/consul/config
  consul2:
    image: consul:latest
    container_name: consul2
    restart: always
    command: agent -server -client=0.0.0.0 -retry-join=consul1 -node=consul2
    volumes:
      - /usr/local/docker_my/consul/consul2/data:/consul/data
      - /usr/local/docker_my/consul/consul2/config:/consul/config
  consul3:
    image: consul:latest
    container_name: consul3
    restart: always
    command: agent -server -client=0.0.0.0 -retry-join=consul1 -node=consul3
    volumes:
      - /usr/local/docker_my/consul/consul3/data:/consul/data
      - /usr/local/docker_my/consul/consul3/config:/consul/config
  consul4:
    image: consul:latest
    container_name: consul4
    restart: always
    ports:
      - 8500:8500
    command: agent -client=0.0.0.0 -retry-join=consul1 -ui -node=client1
    volumes:
      - /usr/local/docker_my/consul/consul4/data:/consul/data
      - /usr/local/docker_my/consul/consul4/config:/consul/config
```
说明：

启动了4个consul，其中consul1 是主节点，consul2、consul3 是子节点。consul4是提供ui服务的。

启动：
`docker-compose up -d`

参数说明：
server模式启动的命令行参数说明：
+ -server：表示当前使用的server模式；如果没有指定，则表示是client模式。
+ -node：指定当前节点在集群中的名称。
+ -config-dir：指定配置文件路径，定义服务的；路径下面的所有.json结尾的文件都被访问；缺省值为：/consul/config。
+ -data-dir： consul存储数据的目录；缺省值为：/consul/data。
+ -datacenter：数据中心名称，缺省值为dc1。
+ -ui：使用consul自带的web UI界面 。
+ -join：加入到已有的集群中。
+ -enable-script-checks： 检查服务是否处于活动状态，类似开启心跳。
+ -bind： 绑定服务器的ip地址。
+ -client： 客户端可访问ip，缺省值为：“127.0.0.1”，即仅允许环回连接。
+ -bootstrap-expect：在一个datacenter中期望的server节点数目，consul启动时会一直等待直到达到这个数目的server才会引导整个集群。这个参数的值在同一个datacenter的所有server节点上必须保持一致。


## consul安装(集群)
首先准备三个节点node1、node2、node3：
+ 10.25.84.163
+ 10.25.84.164
+ 10.25.84.165

### 下载安装包
以linux下安装为例，首先下载安装包，下载地址：`https://www.consul.io/downloads.html`
下载后上传到linux服务器，或者直接在linux上下载，版本可自行替换
```shell
wget https://releases.hashicorp.com/consul/1.7.0/consul_1.7.0_linux_amd64.zip
unzip consul_1.7.0_linux_amd64.zip -d /usr/local/bin
```
### 设置环境变量
```shell
$ vi /etc/profile
export CONSUL_HOME=/usr/local/bin/consul
export PATH=$PATH:CONSUL_HOME

$ source /etc/profile
```
### 验证（三台机）
```shell
$ consul version
Consul v1.7.0
Protocol 2 spoken by default, understands 2 to 3
```

### 启动agent
分别在三台服务器输入以下对应的命令：
```shell
// 启动10.25.84.163
consul agent -server -ui -bootstrap-expect=3 -data-dir=/data/consul -node=server-1 -client=0.0.0.0 -bind=10.25.84.163 -datacenter=dc1

// 启动10.25.84.164，并加入10.25.84.163节点
consul agent -server -ui -bootstrap-expect=3 -data-dir=/data/consul -node=server-2 -client=0.0.0.0 -bind=10.25.84.164 -datacenter=dc1 -join 10.25.84.163

// 启动10.25.84.165，并加入10.25.84.163节点
consul agent -server -ui -bootstrap-expect=3 -data-dir=/data/consul -node=server-3 -client=0.0.0.0 -bind=10.25.84.165 -datacenter=dc1 -join 10.25.84.163

```
#### 启动日志
```shell
[root@localhost ~]# consul agent -server -ui -bootstrap-expect=3 -data-dir=/data/consul -node=server-1 -client=0.0.0.0 -bind=10.25.84.163 -datacenter=dc1
bootstrap_expect > 0: expecting 3 servers
==> Starting Consul agent...
           Version: 'v1.7.0'
           Node ID: '4a60c9bd-472b-01a3-57f4-c74b8ba4d3df'
         Node name: 'server-1'
        Datacenter: 'dc1' (Segment: '<all>')
            Server: true (Bootstrap: false)
       Client Addr: [0.0.0.0] (HTTP: 8500, HTTPS: -1, gRPC: -1, DNS: 8600)
      Cluster Addr: 10.25.84.163 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false, Auto-Encrypt-TLS: false

==> Log data will now stream in as it occurs:

    2020-02-21T11:28:20.323+0800 [INFO]  agent.server.raft: initial configuration: index=0 servers=[]
    2020-02-21T11:28:20.324+0800 [INFO]  agent.server.raft: entering follower state: follower="Node at 10.25.84.163:8300 [Follower]" leader=
    2020-02-21T11:28:20.325+0800 [INFO]  agent.server.serf.wan: serf: EventMemberJoin: server-1.dc1 10.25.84.163
    2020-02-21T11:28:20.326+0800 [INFO]  agent.server.serf.lan: serf: EventMemberJoin: server-1 10.25.84.163
    2020-02-21T11:28:20.326+0800 [INFO]  agent.server: Adding LAN server: server="server-1 (Addr: tcp/10.25.84.163:8300) (DC: dc1)"
    2020-02-21T11:28:20.326+0800 [INFO]  agent.server: Handled event for server in area: event=member-join server=server-1.dc1 area=wan
    2020-02-21T11:28:20.327+0800 [INFO]  agent: Started DNS server: address=0.0.0.0:8600 network=tcp
    2020-02-21T11:28:20.327+0800 [INFO]  agent: Started DNS server: address=0.0.0.0:8600 network=udp
    2020-02-21T11:28:20.328+0800 [INFO]  agent: Started HTTP server: address=[::]:8500 network=tcp
    2020-02-21T11:28:20.328+0800 [INFO]  agent: started state syncer
==> Consul agent running!
...
    2020-02-21T11:29:27.352+0800 [INFO]  agent.server.raft: pipelining replication: peer="{Voter 31174405-571f-f598-ef74-0a9aba59a6a8 10.25.84.165:8300}"
    2020-02-21T11:29:27.352+0800 [WARN]  agent.server.raft: appendEntries rejected, sending older logs: peer="{Voter b3d84299-a458-19bf-2b98-ca9031e6aea4 10.25.84.164:8300}" next=1
    2020-02-21T11:29:27.356+0800 [INFO]  agent.server.raft: pipelining replication: peer="{Voter b3d84299-a458-19bf-2b98-ca9031e6aea4 10.25.84.164:8300}"
    2020-02-21T11:29:27.368+0800 [INFO]  agent.leader: started routine: routine="CA root pruning"
    2020-02-21T11:29:27.368+0800 [INFO]  agent.server: member joined, marking health alive: member=server-1
    2020-02-21T11:29:27.381+0800 [INFO]  agent.server: member joined, marking health alive: member=server-2
    2020-02-21T11:29:27.396+0800 [INFO]  agent.server: member joined, marking health alive: member=server-3
    2020-02-21T11:29:28.609+0800 [INFO]  agent: Synced node info
```
#### 查看集群成员：
```shell
[root@localhost ~]# consul members
Node      Address            Status  Type    Build  Protocol  DC   Segment
server-1  10.25.84.163:8301  alive   server  1.7.0  2         dc1  <all>
server-2  10.25.84.164:8301  alive   server  1.7.0  2         dc1  <all>
server-3  10.25.84.165:8301  alive   server  1.7.0  2         dc1  <all>

```
命令输出显示了集群节点名称、IP端口、健康状态、启动模式、所在数据中心和版本信息。

####  通过HTTP API查看
```shell
curl 10.25.84.163:8500/v1/catalog/nodes
```
返回数据
```shell
[
    {
        "ID":"99f8e10c-edda-bb40-21b4-8719ba851308",
        "Node":"agent-1",
        "Address":"10.25.84.163",
        "Datacenter":"dc1",
        "TaggedAddresses":{
            "lan":"10.25.84.163",
            "lan_ipv4":"10.25.84.163",
            "wan":"10.25.84.163",
            "wan_ipv4":"10.25.84.163"
        },
        "Meta":{
            "consul-network-segment":""
        },
        "CreateIndex":5,
        "ModifyIndex":11
    },
    {
        "ID":"824a86e6-9713-a136-8037-c489222a88e1",
        "Node":"agent-2",
        "Address":"10.25.84.164",
        "Datacenter":"dc1",
        "TaggedAddresses":{
            "lan":"10.25.84.164",
            "lan_ipv4":"10.25.84.164",
            "wan":"10.25.84.164",
            "wan_ipv4":"10.25.84.164"
        },
        "Meta":{
...
```

####  通过DNS接口查看
```shell
[root@localhost ~]$ dig @10.25.84.163 -p 8600 server-1.node.consul

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-9.P2.el7 <<>> @10.25.84.163 -p 8600 server-1.node.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64004
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;server-1.node.consul.		IN	A

;; ANSWER SECTION:
server-1.node.consul.	0	IN	A	10.25.84.163

;; ADDITIONAL SECTION:
server-1.node.consul.	0	IN	TXT	"consul-network-segment="

;; Query time: 1 msec
;; SERVER: 10.25.84.163#8600(10.25.84.163)
;; WHEN: Fri Feb 21 12:47:15 CST 2020
;; MSG SIZE  rcvd: 101

```
其中server-1.node.consul中第一部分需要替换成自己的agent节点名

dig命令如果不存在需要安装下，命令：
`yum -y install bind-utils`

#### 停止agent服务
通过consul leave命令优雅停止服务，我们再打开一个从节点机器终端，运行该命令
```shell
$ consul leave
Graceful leave complete
```



## kratos微服务注册consul
### 步骤一：引入依赖包
```shell
$go get -u github.com/hashicorp/consul/api
$go get -u github.com/go-kratos/kratos/contrib/registry/consul/v2
$go mod tidy
```
### 步骤二：添加配置config.proto
文件：`website/internal/conf/conf.proto`
添加
```proto
message Registry {
  message Consul {
    string address = 1;
    string scheme = 2;
  }
  Consul consul = 1;
}
```
当前目录下重置pb.go文件：
```shell
$ protoc --proto_path=./internal --go_out=paths=source_relative:./internal ./internal/conf/conf.proto

```

### 步骤三：添加consul 配置信息
```yaml
server:
  http:
    addr: 0.0.0.0:8902
    timeout: 1s
  grpc:
    addr: 0.0.0.0:9902
    timeout: 1s
  env:
    logsystem: "/Users/wangdante/D/kugou/hl/hl_service/backendapi/logs/system/"
    loginfo: "/Users/wangdante/D/kugou/hl/hl_service/backendapi/logs/info/"
  #consul 配置信息
  consul:
    address: 127.0.0.1:8500
    scheme: http
```

### 步骤四：添加consul服务
目录：
`website/internal/server/consul.go`
内容如下：
```go
package server

import (
	"github.com/go-kratos/kratos/contrib/registry/consul/v2"
	"github.com/go-kratos/kratos/v2/log"
	"github.com/go-kratos/kratos/v2/registry"
	consulAPI "github.com/hashicorp/consul/api"
	"github.com/spf13/viper"
	"website/internal/conf"
)

// NewRegistrar new an Consul server.
func NewRegistrar(conf *conf.Registry, logger log.Logger) registry.Registrar {

	c := consulAPI.DefaultConfig()

	c.Address = viper.GetString("server.consul.address")
	//c.Address = "127.0.0.1:8500"
	c.Scheme = viper.GetString("server.consul.scheme")
	//c.Scheme = "http"

	cli, err := consulAPI.NewClient(c)
	if err != nil {
		panic(err)
	}
	r := consul.New(cli, consul.WithHealthCheck(false))
	return r
}
```
### 步骤五：依赖注入consul
目录：`website/internal/server/server.go`
```go
var ProviderSet = wire.NewSet(NewGRPCServer, NewHTTPServer, NewRegistrar) //添加服务NewRegistrar
```
### 步骤六：main.go入口文件引入consul
```go
...
//添加 rr registry.Registrar 
//并注册kratos.Registrar
func newApp(logger log.Logger, gs *grpc.Server, hs *http.Server, rr registry.Registrar) *kratos.App {
	return kratos.New(
		kratos.ID(id),
		kratos.Name(Name),
		kratos.Version(Version),
		kratos.Metadata(map[string]string{}),
		kratos.Logger(logger),
		kratos.Server(
			gs,
			hs,
		),
		kratos.Registrar(rr),
	)
}
...
func main() {
...
    //读取配置信息
	var rc conf.Registry
	if err := c.Scan(&rc); err != nil {
		panic(err)
	}
...
    //注入 &rc
	app, cleanup, err := wireApp(bc.Server, &rc, bc.Data, logger)
...
```
### 步骤七：初始化应用registrar
目录：
`website/cmd/website/wire_gen.go`
```go
...
// wireApp init kratos application.
// 添加 registry *conf.Registry
//添加newApp应用
func wireApp(confServer *conf.Server, registry *conf.Registry, confData *conf.Data, logger log.Logger) (*kratos.App, func(), error) {
...
	registrarServer := server.NewRegistrar(registry, logger)
	app := newApp(logger, grpcServer, httpServer, registrarServer)
...
}
```

目录：
`website/cmd/website/wire.go`
```go
...
// wireApp init kratos application.
// 添加*conf.Registry
func wireApp(*conf.Server, *conf.Registry, *conf.Data, log.Logger) (*kratos.App, func(), error) {
	panic(wire.Build(server.ProviderSet, data.ProviderSet, biz.ProviderSet, service.ProviderSet, newApp))
}
```

### 步骤八：启动
```shell
go mod tidy
go get github.com/google/wire/cmd/wire@latest
go generate ./...

kratos run
```
>PS:可能报错：服务名称重复
> main.go ==> id = Name + "-" + uuid.NewString()

### 验证成功
去浏览器查看：
![/images/docImages/cl3.png](/images/docImages/cl3.png)


