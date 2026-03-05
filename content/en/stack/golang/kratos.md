---
author: "karson"
title: "kratos微服务框架"
date: 2022-12-03 00:00:01
description: "一套轻量级 Go 微服务框架，包含大量微服务相关框架及工具"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags: 
- golang
- categories:




---

# kratos

## 优势

| 比对项       | 内容                                                       |
|-----------|----------------------------------------------------------|
| 框架名	      | kratos                                                   |
| 维护公司	     | Bilibli                                                  |
| 项目地址      | https://github.com/go-kratos/kratos                      |
| star数	    | 21.7k                                                    |
| 开源时间	     | 2019年                                                    |
| 服务治理      | 服务注册/发现、负载均衡、熔断、限流、异常恢复、监控、链路跟踪、日志等                      |
| 传输协议	     | gRPC、HTTP                                                |
| 服务发现拓展支持	 | nacos、consul、etcd、polaris、kubernetes、discovery、zookeeper |
| API定义	    | 仅支持Protobuf                                              |
| 框架原则特点    | 简单、通用、高效、稳定、健壮、高性能、扩展性、容错性、工具                            |

## 特性
+ APIS: 协议通信以 HTTP/gRPC 为基础，通过 Protobuf 进行定义
+ Errors: 通过 Protobuf 的 Enum 作为错误码定义，以及工具生成判定接口
+ Metadata: 在协议通信 HTTP/gRPC 中，通过 Middleware 规范化服务元信息传递
+ Config: 支持多数据源方式，进行配置合并铺平，通过 Atomic 方式支持动态配置
+ Logger: 标准日志接口，可方便集成三方 log 库，并可通过 fluentd 收集日志
+ Metrics: 统一指标接口，可以实现各种指标系统，默认集成 Prometheus
+ Tracing: 遵循 OpenTelemetry 规范定义，以实现微服务链路追踪
+ Encoding: 支持 Accept 和 Content-Type 进行自动选择内容编码
+ Transport: 通用的 HTTP/gRPC 传输层，实现统一的 Middleware 插件支持
+ Registry: 实现统一注册中心接口，可插件化对接各种注册中心

## kratos-layout方案
kratos官方提供了一套目录结构方案
```shell
  .
├── Dockerfile  
├── LICENSE
├── Makefile  
├── README.md
├── api // 下面维护了微服务使用的proto文件以及根据它们所生成的go文件
│   └── helloworld
│       └── v1
│           ├── error_reason.pb.go
│           ├── error_reason.proto
│           ├── error_reason.swagger.json
│           ├── greeter.pb.go
│           ├── greeter.proto
│           ├── greeter.swagger.json
│           ├── greeter_grpc.pb.go
│           └── greeter_http.pb.go
├── cmd  // 整个项目启动的入口文件
│   └── server
│       ├── main.go
│       ├── wire.go  // 我们使用wire来维护依赖注入
│       └── wire_gen.go
├── configs  // 这里通常维护一些本地调试用的样例配置文件
│   └── config.yaml
├── generate.go
├── go.mod
├── go.sum
├── internal  // 该服务所有不对外暴露的代码，通常的业务逻辑都在这下面，使用internal避免错误引用
│   ├── biz   // 业务逻辑的组装层，类似 DDD 的 domain 层，data 类似 DDD 的 repo，而 repo 接口在这里定义，使用依赖倒置的原则。
│   │   ├── README.md
│   │   ├── biz.go
│   │   └── greeter.go
│   ├── conf  // 内部使用的config的结构定义，使用proto格式生成
│   │   ├── conf.pb.go
│   │   └── conf.proto
│   ├── data  // 业务数据访问，包含 cache、db 等封装，实现了 biz 的 repo 接口。我们可能会把 data 与 dao 混淆在一起，data 偏重业务的含义，它所要做的是将领域对象重新拿出来，我们去掉了 DDD 的 infra层。
│   │   ├── README.md
│   │   ├── data.go
│   │   └── greeter.go
│   ├── server  // http和grpc实例的创建和配置
│   │   ├── grpc.go
│   │   ├── http.go
│   │   └── server.go
│   └── service  // 实现了 api 定义的服务层，类似 DDD 的 application 层，处理 DTO 到 biz 领域实体的转换(DTO -> DO)，同时协同各类 biz 交互，但是不应处理复杂逻辑
│       ├── README.md
│       ├── greeter.go
│       └── service.go
└── third_party  // api 依赖的第三方proto
    ├── README.md
    ├── google
    │   └── api
    │       ├── annotations.proto
    │       ├── http.proto
    │       └── httpbody.proto
    └── validate
        ├── README.md
        └── validate.proto
```

## 安装
### 依赖环境：
1. go
2. protoc
```shell
protoc --version
libprotoc 25.0
```
3. 安装protobuf的go扩展工具 protoc-gen-go
```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go
```
4. 建议开启GO111MODULE
```shell
go env -w GO111MODULE=on
```
5. 安装kratos脚手架
su root
```shell
GOPROXY=https://goproxy.io,direct go install github.com/go-kratos/kratos/cmd/kratos/v2@latest && kratos upgrade

```
6. 安装kratos框架
切换root，因数会把kratos 放到bin目录
`go install github.com/go-kratos/kratos/cmd/kratos/v2@latest`
验证：
```shell
kratos -v
kratos version v2.7.1
```

> PS:把 $GOPATH/bin 加入系统环境变量PATH:
> linux:
> vim /etc/profile
> export PATH="/path/to/$GOPATH/bin:$PATH"
> source /etc/profile
## 创建项目

`kratos new helloworld -r https://gitee.com/go-kratos/kratos-layout.git`
```shell
kratos new helloworld -r https://gitee.com/go-kratos/kratos-layout.git
🚀 Creating service helloworld, layout repo is https://gitee.com/go-kratos/kratos-layout.git, please wait a moment.

正克隆到 '/Users/wangdante/.kratos/repo/gitee.com/go-kratos/kratos-layout@main'...

CREATED helloworld/.gitignore (552 bytes)
CREATED helloworld/Dockerfile (459 bytes)
CREATED helloworld/LICENSE (1066 bytes)
CREATED helloworld/Makefile (2525 bytes)
CREATED helloworld/README.md (1062 bytes)
CREATED helloworld/api/helloworld/v1/error_reason.pb.go (4991 bytes)
CREATED helloworld/api/helloworld/v1/error_reason.proto (290 bytes)
CREATED helloworld/api/helloworld/v1/greeter.pb.go (8074 bytes)
CREATED helloworld/api/helloworld/v1/greeter.proto (678 bytes)
CREATED helloworld/api/helloworld/v1/greeter_grpc.pb.go (3560 bytes)
CREATED helloworld/api/helloworld/v1/greeter_http.pb.go (2139 bytes)
CREATED helloworld/cmd/helloworld/main.go (1744 bytes)
CREATED helloworld/cmd/helloworld/wire.go (607 bytes)
CREATED helloworld/cmd/helloworld/wire_gen.go (1069 bytes)
CREATED helloworld/configs/config.yaml (266 bytes)
CREATED helloworld/go.mod (1053 bytes)
CREATED helloworld/go.sum (18535 bytes)
CREATED helloworld/internal/biz/README.md (6 bytes)
CREATED helloworld/internal/biz/biz.go (128 bytes)
CREATED helloworld/internal/biz/greeter.go (1236 bytes)
CREATED helloworld/internal/conf/conf.pb.go (20782 bytes)
CREATED helloworld/internal/conf/conf.proto (761 bytes)
CREATED helloworld/internal/data/README.md (7 bytes)
CREATED helloworld/internal/data/data.go (473 bytes)
CREATED helloworld/internal/data/greeter.go (835 bytes)
CREATED helloworld/internal/server/grpc.go (826 bytes)
CREATED helloworld/internal/server/http.go (831 bytes)
CREATED helloworld/internal/server/server.go (150 bytes)
CREATED helloworld/internal/service/README.md (10 bytes)
CREATED helloworld/internal/service/greeter.go (688 bytes)
CREATED helloworld/internal/service/service.go (136 bytes)
CREATED helloworld/openapi.yaml (1130 bytes)
CREATED helloworld/third_party/README.md (14 bytes)
CREATED helloworld/third_party/errors/errors.proto (411 bytes)
CREATED helloworld/third_party/google/api/annotations.proto (1051 bytes)
CREATED helloworld/third_party/google/api/client.proto (3395 bytes)
CREATED helloworld/third_party/google/api/field_behavior.proto (3011 bytes)
CREATED helloworld/third_party/google/api/http.proto (15140 bytes)
CREATED helloworld/third_party/google/api/httpbody.proto (2671 bytes)
CREATED helloworld/third_party/google/protobuf/any.proto (5909 bytes)
CREATED helloworld/third_party/google/protobuf/api.proto (7734 bytes)
CREATED helloworld/third_party/google/protobuf/compiler/plugin.proto (8754 bytes)
CREATED helloworld/third_party/google/protobuf/descriptor.proto (38497 bytes)
CREATED helloworld/third_party/google/protobuf/duration.proto (4895 bytes)
CREATED helloworld/third_party/google/protobuf/empty.proto (2429 bytes)
CREATED helloworld/third_party/google/protobuf/field_mask.proto (8185 bytes)
CREATED helloworld/third_party/google/protobuf/source_context.proto (2341 bytes)
CREATED helloworld/third_party/google/protobuf/struct.proto (3779 bytes)
CREATED helloworld/third_party/google/protobuf/timestamp.proto (6459 bytes)
CREATED helloworld/third_party/google/protobuf/type.proto (6126 bytes)
CREATED helloworld/third_party/google/protobuf/wrappers.proto (4042 bytes)
CREATED helloworld/third_party/openapi/v3/annotations.proto (2195 bytes)
CREATED helloworld/third_party/openapi/v3/openapi.proto (22082 bytes)
CREATED helloworld/third_party/validate/README.md (81 bytes)
CREATED helloworld/third_party/validate/validate.proto (31270 bytes)

🍺 Project creation succeeded helloworld
💻 Use the following command to start the project 👇:

$ cd helloworld
$ go generate ./...
$ go build -o ./bin/ ./... 
$ ./bin/helloworld -conf ./configs

                        🤝 Thanks for using Kratos
        📚 Tutorial: https://go-kratos.dev/docs/getting-started/start

```
 -r 指定源

### 整理依赖关系
```shell
go mod tidy
go mod tidy: go.mod file indicates go 1.18, but maximum supported version is 1.17
```
切换到go 1.18版本

```shell
go mod tidy
```
### 依赖注入wire
```golang
go get github.com/google/wire/cmd/wire
go generate ./..
// 写入依赖注入
wrote /Users/wangdante/D/kugou/verify-code/cmd/verify-code/wire_gen.go 
```

### 添加服务
共用 go.mod ，大仓模式
```golang
kratos new helloworld
cd helloworld
kratos new app/user --nomod

├── app
│   └── user
│       ├── Dockerfile
│       ├── Makefile
│       ├── cmd
│       │   └── user
│       │       ├── main.go
│       │       ├── wire.go
│       │       └── wire_gen.go
│       ├── configs
│       │   └── config.yaml
│       ├── internal
│       │   ├── biz
│       │   │   ├── biz.go
│       │   │   └── greeter.go
│       │   ├── conf
│       │   │   ├── conf.pb.go
│       │   │   └── conf.proto
│       │   ├── data
│       │   │   ├── data.go
│       │   │   └── greeter.go
│       │   ├── server
│       │   │   ├── grpc.go
│       │   │   ├── http.go
│       │   │   └── server.go
│       │   └── service
│       │       ├── greeter.go
│       │       └── service.go
│       └── openapi.yaml
```

####  一、添加Proto文件
> kratos-layout 项目中对 proto 文件进行了版本划分，放在了 v1 子目录下
```golang 
kratos proto add api/helloworld/v1/demo.proto
```
####  二、生成Proto代码
```shell
# 可以直接通过 make 命令生成
make api

# 或使用 kratos cli 进行生成
kratos proto client api/helloworld/v1/demo.proto
```
会在proto文件同目录下生成:
```shell
api/helloworld/v1/demo.pb.go
api/helloworld/v1/demo_grpc.pb.go
# 注意 http 代码只会在 proto 文件中声明了 http 时才会生成
api/helloworld/v1/demo_http.pb.go
```
####  三、生成Service代码
通过 proto 文件，可以直接生成对应的 Service 实现代码：
使用 -t 指定生成目录
```shell
kratos proto server api/helloworld/v1/demo.proto -t internal/service
```
输出：
internal/service/demo.go





### 规范代码 (举例verify-code项目)
使用 internal/data 目录 完成数据(数据库，缓存，文件，OSS，云服务器)的操作
对redis的操作，就属于此类
##### 步骤一：在/internal/data/data.go中完成redis客户端的初始化
##### 步骤二：使用配置文件，完成redis服务器信息的设置
```golang
// ProviderSet is data providers.
var ProviderSet = wire.NewSet(NewData, NewGreeterRepo)

// Data .
type Data struct {
	// TODO wrapped database client
	Rdb *redis.Client
}

// NewData .
func NewData(c *conf.Data, logger log.Logger) (*Data, func(), error) {
	data := &Data{}
	//初始化Rdb
	//连接redis,使用服务的配置，c就是解析之后的变量
	redisURL := fmt.Sprintf("redis://%s/1?dial_timeout=%d", c.Redis.Addr, 1)
	options, err := redis.ParseURL(redisURL)
	if err != nil {
		data.Rdb = nil
	}
	// new client 不会立即连接，建立客户端，需要执行命令时才会连接
	data.Rdb = redis.NewClient(options)

	cleanup := func() {
		//清理了redis连接
		_ = data.Rdb.Close()
		log.NewHelper(logger).Info("closing the data resources")
	}
	return data, cleanup, nil
}
```
##### 步骤三：创建data中用于完成数据操作的对象(实体)
1. 新建 internal/data/customer.go
2. 定义完成后，为依赖注入 wire 提供 provider
3. 在 internal/data/data.go 中

##### 步骤四：完成设置验证码的业务逻辑代码
更新 internal/data/customer.go 新建
`func (cd CustomerData) SetVerifyCode(telephone, code string, ex int64) error` 方法，用于 完成设置

##### 步骤五：更新 CustomerService 的定义，与 CustomerData 建立关联
`internal/service/customer.go`
修改如下：
```golang
type CustomerService struct {
	pb.UnimplementedCustomerServer
	cd *data.CustomerData //修改
}

func NewCustomerService(cd *data.CustomerData) *CustomerService {
	return &CustomerService{
		cd: cd, //修改
	}
}
```

##### 步骤六：调用 CustomerData 中定义的方法，实现设置验证码缓存的业务逻辑
1. 添加 Req 和 Resp

customer.proto 文件中 添加 Req 和 Resp 结构，生成更新customer.pd.go 文件：
```proto
message GetVerifyCodeReq{}
message GetVerifyCodeResp{}
```
2. 更新customer.pb.go文件：
```shell
kratos proto client api/helloworld/v1/demo.proto
```
3. GetVerifyCode方法



>每次更新ProviderSet后都要重新更新依赖注入
> go generate ./...

查看makefile:
```shell
make help
sudo make init
sudo make api
```
运行项目
```shell
kratos run 
# 修改配置文件 config.yaml 端口
2023/11/21 19:29:48 maxprocs: Leaving GOMAXPROCS=8: CPU quota undefined
DEBUG msg=config loaded: config.yaml format: yaml
INFO ts=2023-11-21T19:29:48+08:00 caller=http/server.go:302 service.id=wangdeMacBook-Pro.local service.name= service.version= trace.id= span.id= msg=[HTTP] server listening on: [::]:8022
INFO ts=2023-11-21T19:29:48+08:00 caller=grpc/server.go:205 service.id=wangdeMacBook-Pro.local service.name= service.version= trace.id= span.id= msg=[gRPC] server listening on: [::]:9022

```

打包项目
```shell
$go build -o ./bin/ ./...
./helloworld 
2023/12/05 11:05:26 maxprocs: Leaving GOMAXPROCS=8: CPU quota undefined
panic: stat ../../configs: no such file or directory
cp -r ../configs ../../

./helloworld 
2023/12/05 11:07:05 maxprocs: Leaving GOMAXPROCS=8: CPU quota undefined
DEBUG msg=config loaded: config.yaml format: yaml
INFO ts=2023-12-05T11:07:05+08:00 caller=http/server.go:317 service.id=wangdeMacBook-Pro.local service.name= service.version= trace.id= span.id= msg=[HTTP] server listening on: [::]:8000
INFO ts=2023-12-05T11:07:05+08:00 caller=grpc/server.go:212 service.id=wangdeMacBook-Pro.local service.name= service.version= trace.id= span.id= msg=[gRPC] server listening on: [::]:9006
INFO ts=2023-12-05T11:07:15+08:00 caller=biz/greeter.go:44 service.id=wangdeMacBook-Pro.local service.name= service.version= trace.id= span.id= msg=CreateGreeter: eric
INFO ts=2023-12-05T11:07:16+08:00 caller=biz/greeter.go:44 service.id=wangdeMacBook-Pro.local service.name= service.version= trace.id= span.id= msg=CreateGreeter: eric

```


## 大集成demo
### 创建服务customer
```shell
kratos new customer -r https://gitee.com/go-kratos/kratos-layout.git  
```

### 整理依赖包
```shell
cd customer
go mod tidy
```

### 依赖注入wire
```shell
go get github.com/google/wire/cmd/wire
// 写入依赖注入
go generate ./...
wire: customer/cmd/customer: wrote /Users/wangdante/D/kugou/kratos_backend/customer/cmd/customer/wire_gen.go
```

### 验证启动

>PS kratos run 的时候报错：
> missing go.sum entry for module providing package golang.org/x/sync/errgroup (imported by github.com/go-kratos/kratos/v2); to add:
> 解决方法： go get golang.org/x/sync/errgroup

修改 configs/config.yaml 端口：
```yaml
server:
  http:
    addr: 0.0.0.0:8600
    timeout: 1s
  grpc:
    addr: 0.0.0.0:9600
```

开启项目：`kratos run` --在customer目录下

验证访问：http://127.0.0.1:8600/helloworld/123

### 添加proto文件
```shell
kratos proto add api/customer/customer.proto
```

### 增加路由代码
vim api/customer/customer.proto 中添加内容如下：
```proto
// 导入包
import "google/api/annotations.proto";
...
//获取验证码
	rpc GetVerifyCode (GetVerifyCodeReq)returns(GetVerifyCodeResp){
		option (google.api.http)={
			get: "/customer/get-verify-code"
		};
	}
...
message GetVerifyCodeReq {
	string Telephone = 1;
}
message GetVerifyCodeResp {
	int64 Code = 1;
	string Message = 2;
	string Data = 3;
}
```

### 增加客户端代码(client)
```shell
kratos proto client api/customer/customer.proto 
```

### 增加服务端代码(server)
```shell
kratos proto server api/customer/customer.proto
#或者指定目录 -t
kratos proto server api/customer/customer.proto -t internal/service
```

### http服务中加customer服务
修改 internal/server/http.go ：内容如下：
```go
import customer2 "customer/api/customer"
...
func NewHTTPServer(c *conf.Server, customerService *service.CustomerService, greeter *service.GreeterService, logger log.Logger) *http.Server {
...
srv := http.NewServer(opts...)
// 注册customer的http的服务
customer2.RegisterCustomerHTTPServer(srv, customerService)
v1.RegisterGreeterHTTPServer(srv, greeter)
...
```

### (grpc服务中加customer服务)
修改 internal/server/grpc.go ：内容如下：
```go
import customer2 "customer/api/customer"
...
func NewGRPCServer(c *conf.Server, customerService *service.CustomerService, greeter *service.GreeterService, logger log.Logger) *grpc.Server {
...
srv := grpc.NewServer(opts...)
// 注册customer的grpc的服务
customer2.RegisterCustomerServer(srv, customerService)
v1.RegisterGreeterServer(srv, greeter)
```

### 修改依赖注入providerSet
修改 internal/service/service.go 内容如下：
```go
var ProviderSet = wire.NewSet(NewCustomerService, NewGreeterService)
```



### 基于wire注入依赖
集成根目录下 kratos_backend 执行
```shell
go get github.com/google/wire/cmd/wire
```

kratos_backend/customer目录下执行：
```shell
go generate ./...
wire: customer/cmd/customer: wrote /Users/wangdante/D/kugou/kratos_backend/customer/cmd/customer/wire_gen.go

```


>PS:报错
> go: cannot find main module, but found .git/config in /Users/wangdante/D/kugou/kratos_backend to create a module there, run:go mod init
>解决方法：
> go mod init customer
> go mod tidy
> 

### 启动
customer目录下执行：
```shell
kratos run
2023/12/15 18:45:29 maxprocs: Leaving GOMAXPROCS=8: CPU quota undefined
DEBUG msg=config loaded: config.yaml format: yaml
INFO ts=2023-12-15T18:45:29+08:00 caller=grpc/server.go:205 service.id=wangdeMacBook-Pro.local service.name= service.version= trace.id= span.id= msg=[gRPC] server listening on: [::]:9600
INFO ts=2023-12-15T18:45:29+08:00 caller=http/server.go:302 service.id=wangdeMacBook-Pro.local service.name= service.version= trace.id= span.id= msg=[HTTP] server listening on: [::]:8600

```
#### 测试http服务
浏览器输入：
http://127.0.0.1:8600/customer/get-verify-code
返回：
```shell
{
"Code": "0",
"Message": "",
"Data": ""
}
```
#### 测试grpc服务
使用apifox工具
1. 步骤一：设置.proto文件
`/Users/wangdante/D/kugou/kratos_backend/customer/api/customer/customer.proto`
2. 步骤二：设置依赖关系目录
把 third_party 目录中的 google复制一份到项目customer根目录下
`/Users/wangdante/D/kugou/kratos_backend/customer`
3. 步骤三：设置环境
开发环境：127.0.0.1:9600
4. 步骤四：请求gRPC并传参
参数：
```shell
{
    "Telephone":"13510116521"
}
```
返回值：
```shell
{
    "Code": "0",
    "Message": "13510116521获取验证码成功",
    "Data": ""
}
```

## IDE加载google/api/annotations.proto
解决代码中google报红的方法：
编辑器设置 GoLand -> Settings -> Languages & Frameworks -> Protocol Buffers -> 添加项目的 third_party 目录
`file:///Users/wangdante/D/kugou/kratos_backend/customer/third_party`