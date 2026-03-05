---
author: "jarvan"
title: "微服务"
date: 2022-12-03 00:00:01
description: "微服务的精髓：分而治之，合而用之"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags: 
- golang
- categories:




---
>PS: <font color="lightgreen">微服务架构 </font> 是一种设计方法;
> 而  <font color="lightgreen">微服务</font> 这是应该指使用这种方法而设计的一个应用。

## 微服务架构
定义：将复杂的系统使用 `组件化` 的方式进行拆分，并使用轻量级 `通讯方式` 进行整合的一种设计方法。

## 微服务
定义：通过这种架构设计方法拆分出来的一个独立的 组件化 的 `小应用` 。

> 精髓："分而治之，合而用之"

### 单体式开发的缺点：
1. 复杂性逐渐变高
2. 技术债务逐渐上升
3. 维护成本大
4. 持续交付周期长
5. 可扩展性差

### 微服务式开发的特点：
1. 单一职责：
不同的服务通过 `"管道"` 方式灵活组合 
2. 轻量级通信：
XML 和 JSON 它和语言无关、平台无关的；学用协议：通信协议：通常基于HTTP
3. 独立性：
高度解耦
4. 进程隔离：
每个服务单独隔离

### 微服务式开发的缺点：
1. 运维要求较高
每个模块出问题时整个个项目运行异常，不好排查问题，运维要求高
2. 分布式的复杂性
3. 接口调整成本高
一旦用户微服务的接口发生大的变动，那么所有依赖它的微服务都要做相应的调整
4. 重复劳动
每个微服务都要一个工具类

### 微服务式开发的优点：
1. 开发简单
没有太多的累赘
2. 快速响应需求变化
能够快速的影响业务的需求变化
3. 随时随地更新
微服务的部署和更新并不会影响 `全局系统` 的正常运行；多实例部署的情况下，每个服务的重启和更新任何时候都可以
4. 系统更加稳定可靠
高可用的分布式环境之中，有配套的 `监控 `和 `调度` 管理机制，并且还可以提供自由伸缩的管理，充分保障了系统的稳定可靠性

## 重要组件
1. <font color="lightgreen">**protobuf** </font>
跨语言，跨平台 通讯方式 protobuf
轻便高效的结构化数据存储格式，平台无关、语言无关、可扩展
2. <font color="lightgreen">**gRPC** </font>
通讯协议 gRPC
3. <font color="lightgreen">**consul** </font>
调度管理服务发现 consul
4. <font color="lightgreen">**micro** </font>
微服务的框架 micro
5. <font color="lightgreen">**docker** </font>
部署 docker

#### 数据交互的格式比较
1. <font color="lightgreen">**json**:</font>
一般的web项目中，最流行的主要还是json，因为浏览器对于json数据支持非常好，有很多内建的函数支持
2. <font color="lightgreen">**xml**:</font>
在webservice中应该最为广泛，但是相比于json，它的数据更冗余，因为需要成对的闭合标签，json使用了键值对的方式，不仅压缩了一定的数据空间，同时也具有可主读性
3. <font color="lightgreen">**protobuf**:</font>
后起之秀，是谷歌开源的一种数据格式，适合高性，对 `响应速度有要求` 的数据传输场景。因为profobuf是二进制数据格式，需要编码和解码。数据本身不具有可读性。因此只能反序列化之后得到真正可读的数据

#### 相对于其它protobuf更具有优势
1. 序列化后体积相比json和XML很小，适合网络传输
2. 支持跨平台多语言
3. 消息格式升级和兼容性还不错
4. 序列化反序列化速度很快，快于json的处理速度

protobuf有如 XML ，不过它 <font color="lightgreen">更小、更快、也更简单</font> 。你可以定义自己的数据结构，然后使用代码生成器生成的代码来读写一个数据结构。
你甚至可以在无需重新部署程序的情况下更新数据结构。只需使用 Protobuf 对数据结构进行一次描述，即可利用各种不同语言或从各种不同数据流中对你的结构化数据轻松读写。
它有一个非常棒的特性，即 <font color="lightgreen">"向后"</font>  兼容性好，人们不必破坏已部署的、依靠"老"数据格式的程序就可以对数据结构进行升级。
Protobuf语义更清晰，无需类似XML解析器的东西（因为Protobuf编译器会将.proto文件编译生成对应的数据访问类似对Protobuf数据进行序列化、反序列化操作）。
Protobuf的编程模式比较友好，简单易学，同时它拥有良好的文档和示例，对于喜欢简单事物的人而言，protobuf比其他的技术更加有吸引力。

## Protobuf安装
### mac 安装如下----------

#### 1.Protoc安装
```shell
brew install protobuf
#验证
protoc -h
```


>PS:报错：Error: protobuf: unknown or unsupported macOS version: :dunno
> 执行重置 Homebrew : `brew update-reset`
> 
```shell
brew install protobuf@3.17.3
protoc --version
```
>PS: macos当前系统不支持 autoconf 需要升级
下载地址：https://alpha.gnu.org/pub/gnu/autoconf/autoconf-2.72c.tar.gz
```shell
压缩 -> 进入目录
./configure
make  # ->提示要升级M4  ，去下载最新版本：http://ftp.gnu.org/gnu/m4/m4-latest.tar.gz
sudo make install
autoconf --version
autoconf (GNU Autoconf) 2.71
```

>PS: 升级M4的操作步骤如下：
```shell
tar -zxvf m4-1.4.19
cd m4-1.4.19
./configure
make
sudo make install
```

#### 2.Protoc-gen-go的安装 
```shell
# 首先你需要将GOPATH添加到PATH中；Mac中 在终端输入 env 可以查看环境变量；
# 目前Mac默认的终端是zsh，所以需要 编辑 HOME 下的 .zshrc 文件
vim .zshrc

# vim 在输入法为英文的状态下，按i进入编辑模式，将下边内容添加到文件中

export GOPATH=$HOME/go

export PATH=$GOPATH/bin:$PATH

# 然后 按 : 输入wq ，保存退出。 重载一下 .zshrc 文件

source .zshrc

# 然后在终端执行下方命令

go install github.com/golang/protobuf/protoc-gen-go@latest
```
主要的命令 `go install github.com/golang/protobuf/protoc-gen-go@latest`

#### 3.根据proto文件生成对应的GO代码
执行编译：
`protoc --go_out=plugins=grpc:./ *.proto #添加grpc插件`
更新一下没有的依赖包：
```shell
go get -v -u google.golang.org/grpc
```

```shell
# 当你写好对应的Proto文件后
# 在终端 cd 到 proto 文件的目录，然后执行 下方的命令
protoc -I . hello.proto --go_out=plugins=grpc:.

# 在此间我遇到一个问题，在 proto文件中 option的问题 不能直接用 路径不能用单独用 . 至少要有个 /
option go_package= "./;proto";

# 如此就成功通过proto文件生成go代码

# PS：补充个知识
option go_package = "ofc_app;pb_ofc_app_v1";
# option go_package表示生成的go文件的存放地址和包名，分号前是地址，分号后是包名。 
```


### Linux 安装如下----------
#### 一、下载 protobuf:
+ 方式一：（本人使用下载zip）
#地址：https://github.com/protocolbuffers/protobuf
本人用的 protobuf-3.0.x.zip
```shell
git clone https://github.com/protocolbuffers/protobuf.git

```
+ 方式二：
#地址：https://github.com/protocolbuffers/protobuf/releases

####  二、安装依赖库
```shell
$ sudo apt-get install autoconf automake libtool curl make g++ unzip libffi-dev -y
```
####  三、安装
```shell
cd protobuf/
./autogen.sh
./configure
make
sudo make install
#刷新共享库，很重要的一步
sudo ldconfig 
#安装时候会比较卡
#成功后验证
protoc -h
```
####  四、获取proto包
#GO 语言的proto API接口
#修改国内镜像
>PS:报错超时
> `go env -w GO111MODULE=on`
> `go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct` 
> 或 `go env -w GOPROXY=https://goproxy.cn,direct`
> 有时会报错:
> `warning: go env -w GOPROXY=... does not override conflicting OS environment variable`
> 解决方法：
> 重设一下 GOPROXY 既可
> <font color="lightgreen">unset GOPROXY</font>

go get命令 参数：
+ **-u**  : 强制使用网络去更新包和它的依赖包
+ **-v**  : 显示执行的命令
+ **-x**  : 显示用到的命令
+ **-d** : 只下载不安装
+ **-f** : 只有在你包含了 -u 参数的时候才有效，不让 -u 去验证 import 中的每一个都已经获取了，这对于本地 fork 的包特别有用 
+ **-fix** : 在获取源码之后先运行 fix，然后再去做其他的事情 
+ **-t** : 同时也下载需要为运行测试所需要的包
```shell
$ go get -v -u github.com/golang/protobuf/proto
```
### 五、安装protoc-gen-go插件
它是一个go 程序，编译它之后将可执行文件复制到/bin目录

```shell
#安装
 go get -v -u github.com/golang/protobuf/protoc-gen-go

#复制下载的内容
 mkdir -p /go/src/github.com/golang/
 cp -r /go/pkg/mod/github.com/golang/protobuf\@v1.5.3/ /go/src/github.com/golang/
 mv protobuf\@v1.5.3/ protobuf/
#编译
cd $GOPATH/src/github.com/golang/protobuf/protoc-gen-go/
go build

#将生成的 protoc-gen-go 可执行文件，放在/bin目录下
sudo cp protoc-gen-go /bin/

#验证直接执行命令
protoc-gen-go
```

## Protobuf的语法
`。proto` 文件
定义一个消息类型：
```shell
syntax = "proto3" ;

message PandaRequest{
  string name = 1 ; //名字
  int32 shengao = 2 ; //
  repeated int32 tizhong = 3 ;
}

message PandaResponse{
  int32 error = 1 ; //错误号
  string ermessage = 2 ; //错误信息
}
```
## Protobuf编译器
### 1.新建.proto文件：
```shell
mkdir -p /go/src/myproto/proto1/
cd /go/src/myproto/proto1/
vim test1.proto
```
### 2.编辑.proto文件
内容如下：
```go
syntax = "proto3";
option go_package = "./"; // 指定生成的go文件所在path
message PandaRequest{
    string name = 1;
    int32 shengao = 2;
    repeated int32 tizhong = 3;
}
message PandaResponse{
    int32 error = 1;
    string message = 2;
}
```
### 3.编译对应语言的脚本
调用编译器：
```shell
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR path/to/file.proto
# 或者
protoc --go_out=./  *.proto
```
说明：
1. IMPORT_PATH ：
声明了一个 `.proto` 文件所在的解析 `import` 具体目录。如果忽略则使用当前目录。 如果有多个目录则可以多次调用 `--proto_path` ,它们将会顺序的被访问并执行导入。<font color="lightgreen">-I=IMPORT_PATH</font> 是 `--proto_path` 的简化形式。
2. cpp_out ：
输出路径。 `--cpp_out` 在目标目录DST_DIR中产生C++代码。 `--python_out` 在目标目录DST_DIR中产生python代码。`--go_out` 在目标目录DST_DIR中产生GO代码。

### 4.验证并查看脚本
当前目录生产新的脚本：
`test1.pb.go`
内容是go语言语法：
```go                                                     
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
//      protoc-gen-go v1.26.0
//      protoc        (unknown)
// source: test1.proto

package __

import (
        protoreflect "google.golang.org/protobuf/reflect/protoreflect"
        protoimpl "google.golang.org/protobuf/runtime/protoimpl"
        reflect "reflect"
        sync "sync"
)

const (
        // Verify that this generated code is sufficiently up-to-date.
        _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
        // Verify that runtime/protoimpl is sufficiently up-to-date.
        _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)
-- VISUAL --                                                          48        1,1           Top
```

| .proto Type | C++ Type | Python Type | Go Type |
|-------------|----------|-------------|---------|
| double      | double   | float       | float64 |
| float       | float    | float       | float32 |
| int32       | int32    | int         | int32   |
| bool        | bool     | bool        | bool    |
| string      | string   | str/unicode | string  |
| bytes       | string   | str         | []byte  |

### 有默认值
### 其他消息类型
要在 `.proto` 文件中指定，例如：
```proto
message PersonInfo{
  message Person{
    string name = 1;
    int32 shengo = 2;
    repeated int32 tizhong = 3;
  }
  repeated Person info =1;
}
```
其它父消息类型的外部使用这个消息类型，使用 `.` 链式形式，如：
```proto
message PersonMessage{
    PersonInfo.Person info = 1;
}
```
## 定义服务(Service)
```shell
service SearchService{
  //rpc 服务的函数名 (传入参数) 返回 (返回参数)
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

## gRPC
最直观的使用 `protocol buffer` 的RPC系统是 gRPC

gRPC 是一个高性能、开源和通用的 RPC 框架，面向移动和HTTP/2设计。
特性：双向流、流控、头部压缩、单TCP连接上的多复用请求等待，对移动设备有好，更省电、节省空间占用。
### 测试
#### 1.生成go包
测试生成的proto 的go包
```shell
mkdir -p /go/src/myproto/prototext
cd /go/src/myproto/prototext
vim text.proto
```
内容如下：
```proto
syntax = "proto3";
option go_package = "./";
package prototext;
message Test{
    string name = 1;
    repeated int32 tizhong = 2;
    int32 shengao = 3;
    string motto = 4;
}  
```
#### 2.测试go包
```shell
mkdir -p /go/src/myproto/test
cd /go/src/myproto/test
vim test.go
```
test.go内容：
```go
package main

import(
        "myproto/prototext"
        "fmt"
        "github.com/golang/protobuf/proto"
)

func main(){
   text:=&prototext.Test{
      Name:"wjb",
      Tizhong:[]int32{120,125,160,166},
      Shengao:181,
      Motto:"good study!",
   }

   fmt.Println(text)

   data,err:=proto.Marshal(text)
   if err!=nil{
       fmt.Println("failed")
   }

   fmt.Println(data)

   newtext:=&prototext.Test{}

   proto.Unmarshal(data.newtext)
   if err!=nil{
       fmt.Println("failed")
   }  
   fmt.Println(newtext)                                                                             
   fmt.Println(newtext.Name)                                                                             
   fmt.Println(newtext.Shengao)   
}
```

### RPC
rpc包提供了通过网络或其他I/O连接对一个对象的导出方法的访问。
服务端注册一个对象，使它作为一个服务被暴露，服务的名字是该对象的类型名。
注册之后，对象的导出方法就可以被远程访问。
服务端可以注册多个不同类型的对象（服务），但注册具有相同类型的多个对象是错误的。

只有满足如下标准的方法才能用于远程访问，其余方法会被忽略：
+ 方法是导出的 (首字母大写)
+ 方法有两个参数，都是导出类型或内建类型
+ 方法的第二个参数是指针
+ 方法只有一个error接口类型的返回值

事实上，方法必须看起来像这样：
```go
func (t *T) MethodName(argType T1 , replyType *T2) error
```

#### 实例(服务端+客户端)
```shell
rpc
    ├── client.go
    └── server.go
```
server.go 内容如下：
```go
package main

import (
	"fmt"
	"io"
	"net"
	"net/http"
	"net/rpc"
)

func pandatext(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "hello dante hello panda")
}

/*
+ 方法是导出的 (首字母大写)
+ 方法有两个参数，都是导出类型或内建类型
+ 方法的第二个参数是指针
+ 方法只有一个error接口类型的返回值
*/
type Panda int

//函数关键字 (对象) 函数名 (对端发送过来的内容 ，返回给对端的内容) 错误返回值
func (p *Panda) Getinfo(artType int, replyType *int) error {
	fmt.Println("打印对端发送过来的内容：", artType)

	//修改内容値
	*replyType = artType + 1230

	return nil
}

func main() {
	//页面的请求
	http.HandleFunc("/panda", pandatext)

	//-1-服务端注册一个对象
	pd := new(Panda)
	rpc.Register(pd)
	rpc.HandleHTTP()

	ln, err := net.Listen("tcp", ":10086")
	if err != nil {
		fmt.Println("网络错误")
	}
	http.Serve(ln, nil)
}
```

运行server.go：
```shell
go run server.go
打印对端发送过来的内容： 10086
```

client.go 内容如下：
```go
package main

import (
	"fmt"
	"net/rpc"
)

func main() {
	//建立网络连接
	cli, err := rpc.DialHTTP("tcp", "127.0.0.1:10086")
	if err != nil {
		fmt.Println("网络连接失败")
	}

	var pd int
	/*
		func (client *Client) Call(serviceMethod string, args any, reply any) error
	*/
	//-2-连接服务
	err = cli.Call("Panda.Getinfo", 10086, &pd)
	if err != nil {
		fmt.Println("打call失败")
	}
	fmt.Println("最后的值为：", pd)
}

```
运行client.go：
```shell
go run client.go
最后的值为; 11316
```

### gRPC定义服务
```shell
myproto
└── myproto.proto
```
myproto.proto 的内容：
```proto
syntax = "proto3";

package myproto;

//定义服务
service HelloServer{
    //一个打招呼的函数
    rpc Sayhello(HelloReq)returns(HelloRsp){}
    //一个说名字的函数
    rpc Sayname(NameReq)returns(NameRsp){}
}

//客户端发送给服务端
message HelloReq{
    string name = 1;
}
//服务端返回给客户端
message HelloRsp{
    string msg = 1;
}


//客户端发送给服务端
message NameReq{
    string name = 1;
}
//服务端返回给客户端
message NameRsp{
    string msg = 1;
}

```
编译并生成文件：
```shell
protoc --go_out=./ *.proto #不加grpc插件
protoc --go_out=plugins=grpc:./ *.proto #添加grpc插件
```

#### grpc示例
##### 1. 步骤一：在定义proto，并编译

a. 新增proto文件

```shell
echo $GOPATH # /Users/wangdante/go01

# 新建目录newgrpc
mkdir /Users/wangdante/go01/newgrpc

# 新建proto文件
cd /Users/wangdante/go01/newgrpc
touch newgrpc.proto
```

b. newgrpc.proto 内容如下：

```proto
syntax = "proto3";
option go_package = "./";
package newgrpc;

//定义服务
service HisServer{
    //一个打招呼的函数
    rpc Sayhis(HiReqs)returns(HiRsps){}
    //一个说名字的函数
    rpc Saynas(NaReqs)returns(NaRsps){}
}

//客户端发送给服务端
message HiReqs{
    string name = 1;
}
//服务端返回给客户端
message HiRsps{
    string msg = 1;
}


//客户端发送给服务端
message NaReqs{
    string name = 1;
}
//服务端返回给客户端
message NaRsps{
    string msg = 1;
}

```

c. 编译生成对应语言的go文件 newgrpc.pb.go
```shell
protoc --go_out=plugins=grpc:./ *.proto
newgrpc.pb.go
```

d. newgrpc.pb.go 内容如下：
```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.26.0
// 	protoc        v4.25.0
// source: newgrpc.proto

package __

import (
	context "context"
	grpc "google.golang.org/grpc"
	codes "google.golang.org/grpc/codes"
	status "google.golang.org/grpc/status"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
)

const (
	// Verify that this generated code is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
	// Verify that runtime/protoimpl is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

//客户端发送给服务端
type HiReqs struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Name string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
}

func (x *HiReqs) Reset() {
	*x = HiReqs{}
	if protoimpl.UnsafeEnabled {
		mi := &file_newgrpc_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *HiReqs) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*HiReqs) ProtoMessage() {}

func (x *HiReqs) ProtoReflect() protoreflect.Message {
	mi := &file_newgrpc_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use HiReqs.ProtoReflect.Descriptor instead.
func (*HiReqs) Descriptor() ([]byte, []int) {
	return file_newgrpc_proto_rawDescGZIP(), []int{0}
}

func (x *HiReqs) GetName() string {
	if x != nil {
		return x.Name
	}
	return ""
}

//服务端返回给客户端
type HiRsps struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Msg string `protobuf:"bytes,1,opt,name=msg,proto3" json:"msg,omitempty"`
}

func (x *HiRsps) Reset() {
	*x = HiRsps{}
	if protoimpl.UnsafeEnabled {
		mi := &file_newgrpc_proto_msgTypes[1]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *HiRsps) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*HiRsps) ProtoMessage() {}

func (x *HiRsps) ProtoReflect() protoreflect.Message {
	mi := &file_newgrpc_proto_msgTypes[1]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use HiRsps.ProtoReflect.Descriptor instead.
func (*HiRsps) Descriptor() ([]byte, []int) {
	return file_newgrpc_proto_rawDescGZIP(), []int{1}
}

func (x *HiRsps) GetMsg() string {
	if x != nil {
		return x.Msg
	}
	return ""
}

//客户端发送给服务端
type NaReqs struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Name string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
}

func (x *NaReqs) Reset() {
	*x = NaReqs{}
	if protoimpl.UnsafeEnabled {
		mi := &file_newgrpc_proto_msgTypes[2]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *NaReqs) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*NaReqs) ProtoMessage() {}

func (x *NaReqs) ProtoReflect() protoreflect.Message {
	mi := &file_newgrpc_proto_msgTypes[2]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use NaReqs.ProtoReflect.Descriptor instead.
func (*NaReqs) Descriptor() ([]byte, []int) {
	return file_newgrpc_proto_rawDescGZIP(), []int{2}
}

func (x *NaReqs) GetName() string {
	if x != nil {
		return x.Name
	}
	return ""
}

//服务端返回给客户端
type NaRsps struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Msg string `protobuf:"bytes,1,opt,name=msg,proto3" json:"msg,omitempty"`
}

func (x *NaRsps) Reset() {
	*x = NaRsps{}
	if protoimpl.UnsafeEnabled {
		mi := &file_newgrpc_proto_msgTypes[3]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *NaRsps) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*NaRsps) ProtoMessage() {}

func (x *NaRsps) ProtoReflect() protoreflect.Message {
	mi := &file_newgrpc_proto_msgTypes[3]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use NaRsps.ProtoReflect.Descriptor instead.
func (*NaRsps) Descriptor() ([]byte, []int) {
	return file_newgrpc_proto_rawDescGZIP(), []int{3}
}

func (x *NaRsps) GetMsg() string {
	if x != nil {
		return x.Msg
	}
	return ""
}

var File_newgrpc_proto protoreflect.FileDescriptor

var file_newgrpc_proto_rawDesc = []byte{
	0x0a, 0x0d, 0x6e, 0x65, 0x77, 0x67, 0x72, 0x70, 0x63, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12,
	0x07, 0x6e, 0x65, 0x77, 0x67, 0x72, 0x70, 0x63, 0x22, 0x1c, 0x0a, 0x06, 0x48, 0x69, 0x52, 0x65,
	0x71, 0x73, 0x12, 0x12, 0x0a, 0x04, 0x6e, 0x61, 0x6d, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09,
	0x52, 0x04, 0x6e, 0x61, 0x6d, 0x65, 0x22, 0x1a, 0x0a, 0x06, 0x48, 0x69, 0x52, 0x73, 0x70, 0x73,
	0x12, 0x10, 0x0a, 0x03, 0x6d, 0x73, 0x67, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x6d,
	0x73, 0x67, 0x22, 0x1c, 0x0a, 0x06, 0x4e, 0x61, 0x52, 0x65, 0x71, 0x73, 0x12, 0x12, 0x0a, 0x04,
	0x6e, 0x61, 0x6d, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x6e, 0x61, 0x6d, 0x65,
	0x22, 0x1a, 0x0a, 0x06, 0x4e, 0x61, 0x52, 0x73, 0x70, 0x73, 0x12, 0x10, 0x0a, 0x03, 0x6d, 0x73,
	0x67, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x6d, 0x73, 0x67, 0x32, 0x67, 0x0a, 0x09,
	0x48, 0x69, 0x73, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x12, 0x2c, 0x0a, 0x06, 0x53, 0x61, 0x79,
	0x68, 0x69, 0x73, 0x12, 0x0f, 0x2e, 0x6e, 0x65, 0x77, 0x67, 0x72, 0x70, 0x63, 0x2e, 0x48, 0x69,
	0x52, 0x65, 0x71, 0x73, 0x1a, 0x0f, 0x2e, 0x6e, 0x65, 0x77, 0x67, 0x72, 0x70, 0x63, 0x2e, 0x48,
	0x69, 0x52, 0x73, 0x70, 0x73, 0x22, 0x00, 0x12, 0x2c, 0x0a, 0x06, 0x53, 0x61, 0x79, 0x6e, 0x61,
	0x73, 0x12, 0x0f, 0x2e, 0x6e, 0x65, 0x77, 0x67, 0x72, 0x70, 0x63, 0x2e, 0x4e, 0x61, 0x52, 0x65,
	0x71, 0x73, 0x1a, 0x0f, 0x2e, 0x6e, 0x65, 0x77, 0x67, 0x72, 0x70, 0x63, 0x2e, 0x4e, 0x61, 0x52,
	0x73, 0x70, 0x73, 0x22, 0x00, 0x42, 0x04, 0x5a, 0x02, 0x2e, 0x2f, 0x62, 0x06, 0x70, 0x72, 0x6f,
	0x74, 0x6f, 0x33,
}

var (
	file_newgrpc_proto_rawDescOnce sync.Once
	file_newgrpc_proto_rawDescData = file_newgrpc_proto_rawDesc
)

func file_newgrpc_proto_rawDescGZIP() []byte {
	file_newgrpc_proto_rawDescOnce.Do(func() {
		file_newgrpc_proto_rawDescData = protoimpl.X.CompressGZIP(file_newgrpc_proto_rawDescData)
	})
	return file_newgrpc_proto_rawDescData
}

var file_newgrpc_proto_msgTypes = make([]protoimpl.MessageInfo, 4)
var file_newgrpc_proto_goTypes = []interface{}{
	(*HiReqs)(nil), // 0: newgrpc.HiReqs
	(*HiRsps)(nil), // 1: newgrpc.HiRsps
	(*NaReqs)(nil), // 2: newgrpc.NaReqs
	(*NaRsps)(nil), // 3: newgrpc.NaRsps
}
var file_newgrpc_proto_depIdxs = []int32{
	0, // 0: newgrpc.HisServer.Sayhis:input_type -> newgrpc.HiReqs
	2, // 1: newgrpc.HisServer.Saynas:input_type -> newgrpc.NaReqs
	1, // 2: newgrpc.HisServer.Sayhis:output_type -> newgrpc.HiRsps
	3, // 3: newgrpc.HisServer.Saynas:output_type -> newgrpc.NaRsps
	2, // [2:4] is the sub-list for method output_type
	0, // [0:2] is the sub-list for method input_type
	0, // [0:0] is the sub-list for extension type_name
	0, // [0:0] is the sub-list for extension extendee
	0, // [0:0] is the sub-list for field type_name
}

func init() { file_newgrpc_proto_init() }
func file_newgrpc_proto_init() {
	if File_newgrpc_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_newgrpc_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*HiReqs); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_newgrpc_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*HiRsps); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_newgrpc_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*NaReqs); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_newgrpc_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*NaRsps); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
	}
	type x struct{}
	out := protoimpl.TypeBuilder{
		File: protoimpl.DescBuilder{
			GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
			RawDescriptor: file_newgrpc_proto_rawDesc,
			NumEnums:      0,
			NumMessages:   4,
			NumExtensions: 0,
			NumServices:   1,
		},
		GoTypes:           file_newgrpc_proto_goTypes,
		DependencyIndexes: file_newgrpc_proto_depIdxs,
		MessageInfos:      file_newgrpc_proto_msgTypes,
	}.Build()
	File_newgrpc_proto = out.File
	file_newgrpc_proto_rawDesc = nil
	file_newgrpc_proto_goTypes = nil
	file_newgrpc_proto_depIdxs = nil
}

// Reference imports to suppress errors if they are not otherwise used.
var _ context.Context
var _ grpc.ClientConnInterface

// This is a compile-time assertion to ensure that this generated file
// is compatible with the grpc package it is being compiled against.
const _ = grpc.SupportPackageIsVersion6

// HisServerClient is the client API for HisServer service.
//
// For semantics around ctx use and closing/ending streaming RPCs, please refer to https://godoc.org/google.golang.org/grpc#ClientConn.NewStream.
type HisServerClient interface {
	//一个打招呼的函数
	Sayhis(ctx context.Context, in *HiReqs, opts ...grpc.CallOption) (*HiRsps, error)
	//一个说名字的函数
	Saynas(ctx context.Context, in *NaReqs, opts ...grpc.CallOption) (*NaRsps, error)
}

type hisServerClient struct {
	cc grpc.ClientConnInterface
}

func NewHisServerClient(cc grpc.ClientConnInterface) HisServerClient {
	return &hisServerClient{cc}
}

func (c *hisServerClient) Sayhis(ctx context.Context, in *HiReqs, opts ...grpc.CallOption) (*HiRsps, error) {
	out := new(HiRsps)
	err := c.cc.Invoke(ctx, "/newgrpc.HisServer/Sayhis", in, out, opts...)
	if err != nil {
		return nil, err
	}
	return out, nil
}

func (c *hisServerClient) Saynas(ctx context.Context, in *NaReqs, opts ...grpc.CallOption) (*NaRsps, error) {
	out := new(NaRsps)
	err := c.cc.Invoke(ctx, "/newgrpc.HisServer/Saynas", in, out, opts...)
	if err != nil {
		return nil, err
	}
	return out, nil
}

// HisServerServer is the server API for HisServer service.
type HisServerServer interface {
	//一个打招呼的函数
	Sayhis(context.Context, *HiReqs) (*HiRsps, error)
	//一个说名字的函数
	Saynas(context.Context, *NaReqs) (*NaRsps, error)
}

// UnimplementedHisServerServer can be embedded to have forward compatible implementations.
type UnimplementedHisServerServer struct {
}

func (*UnimplementedHisServerServer) Sayhis(context.Context, *HiReqs) (*HiRsps, error) {
	return nil, status.Errorf(codes.Unimplemented, "method Sayhis not implemented")
}
func (*UnimplementedHisServerServer) Saynas(context.Context, *NaReqs) (*NaRsps, error) {
	return nil, status.Errorf(codes.Unimplemented, "method Saynas not implemented")
}

func RegisterHisServerServer(s *grpc.Server, srv HisServerServer) {
	s.RegisterService(&_HisServer_serviceDesc, srv)
}

func _HisServer_Sayhis_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
	in := new(HiReqs)
	if err := dec(in); err != nil {
		return nil, err
	}
	if interceptor == nil {
		return srv.(HisServerServer).Sayhis(ctx, in)
	}
	info := &grpc.UnaryServerInfo{
		Server:     srv,
		FullMethod: "/newgrpc.HisServer/Sayhis",
	}
	handler := func(ctx context.Context, req interface{}) (interface{}, error) {
		return srv.(HisServerServer).Sayhis(ctx, req.(*HiReqs))
	}
	return interceptor(ctx, in, info, handler)
}

func _HisServer_Saynas_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
	in := new(NaReqs)
	if err := dec(in); err != nil {
		return nil, err
	}
	if interceptor == nil {
		return srv.(HisServerServer).Saynas(ctx, in)
	}
	info := &grpc.UnaryServerInfo{
		Server:     srv,
		FullMethod: "/newgrpc.HisServer/Saynas",
	}
	handler := func(ctx context.Context, req interface{}) (interface{}, error) {
		return srv.(HisServerServer).Saynas(ctx, req.(*NaReqs))
	}
	return interceptor(ctx, in, info, handler)
}

var _HisServer_serviceDesc = grpc.ServiceDesc{
	ServiceName: "newgrpc.HisServer",
	HandlerType: (*HisServerServer)(nil),
	Methods: []grpc.MethodDesc{
		{
			MethodName: "Sayhis",
			Handler:    _HisServer_Sayhis_Handler,
		},
		{
			MethodName: "Saynas",
			Handler:    _HisServer_Saynas_Handler,
		},
	},
	Streams:  []grpc.StreamDesc{},
	Metadata: "newgrpc.proto",
}

```
##### 2. 步骤二：在上级目录新建go.mod项目(方面go get XXX)
生成对应的go编译文件中部分依赖包没有需要下载：
```shell
go get -v -u google.golang.org/grpc
```

##### 3. 步骤三：新建test目录测试服务端+客户端
```shell
 tree
.
├── client.go
└── server.go
```
服务端：server.go 内容如下：
```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	pd "mymod/newgrpc"
	"net"
)

type server struct {
}

func (this *server) Sayhis(ctx context.Context, in *pd.HiReqs) (out *pd.HiRsps, err error) {
	//return nil, status.Errorf(codes.Unimplemented, "method Sayhis not implemented")
	return &pd.HiRsps{Msg: "hello " + in.Name}, nil
}
func (this *server) Saynas(ctx context.Context, in *pd.NaReqs) (out *pd.NaRsps, err error) {
	//return nil, status.Errorf(codes.Unimplemented, "method Saynas not implemented")
	return &pd.NaRsps{Msg: "早上好," + in.Name}, nil
}

func main() {
	ln, err := net.Listen("tcp", ":10086")
	if err != nil {
		fmt.Println("网络错误", err)
	}

	//创建grpc的服务
	srv := grpc.NewServer()

	//注册服务
	pd.RegisterHisServerServer(srv, &server{})

	//等待网络连接
	err = srv.Serve(ln)
	if err != nil {
		fmt.Println("网络错误", err)
	}
}
```

客户端：client.go 内容如下：
```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	pd "mymod/newgrpc"
)

func main() {
	conn, err := grpc.Dial("127.0.0.1:10086", grpc.WithInsecure())
	if err != nil {
		fmt.Println("网络异常", err)
	}
	//延时关闭网络
	defer conn.Close()
	//获得grpc句柄
	c := pd.NewHisServerClient(conn)
	//通过句柄调用函数
	res, err := c.Sayhis(context.Background(), &pd.HiReqs{Name: "大王"})
	if err != nil {
		fmt.Println("Sayhis服务调用失败", err)
	}
	fmt.Println("调用 Sayhis 的返回：", res.Msg)

	rel, err := c.Saynas(context.Background(), &pd.NaReqs{Name: "周杰伦"})
	if err != nil {
		fmt.Println("Saynas 服务调用失败", err)
	}
	fmt.Println("调用 Saynas 的返回：", rel.Msg)
}
```

##### 4. 步骤四：客户端调用服务端
进入test目录分别 开启 服务端、客户端
```shell
go run server.go
```

```shell
go run client.go
调用 Sayhis 的返回： hello 大王
调用 Saynas 的返回： 早上好,周杰伦
```
