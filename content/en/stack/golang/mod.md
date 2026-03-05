---
author: "karson"
title: "GO Modules依赖管理"
date: 2022-12-02 00:00:01
description: "依赖包管理是一个非常重要的内容，依赖包处理不好，就会导致编译失败"
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

## Go Modules 简介
Go Modules 是 Go 官方推出的一个 Go 包管理方案
### 特性
+ 可以使包的管理更加简单
+ 支持版本管理
+ 允许同一个模块多个版本共存
+ 可以校验依赖包的哈希值，确保包的一致性，增加安全性
+ 内置在几乎所有的 go 命令中，包括go get、go build、go install、go run、go test、go list等命令
+ 具有 Global Caching 特性，不同项目的相同模块版本，只会在服务器上缓存一份

>PS:在 Go1.14 版本以及之后的版本，Go 官方建议在生产环境中使用 Go Modules。

## Go1.5 版本前：GOPATH
在 Go1.5 版本之前，没有版本控制，所有的依赖包都放在 GOPATH 下。采用这种方式，无法实现包的多版本管理，并且包的位置只能局限在 GOPATH 目录下。如果 A 项目和 B 项目用到了同一个 Go 包的不同版本，这时候只能给每个项目设置一个 GOPATH，将对应版本的包放在各自的 GOPATH 目录下，切换项目目录时也需要切换 GOPATH，这些都增加了开发和实现的复杂度

## Go1.5 版本：Vendoring
Go1.5 推出了 vendor 机制，并在 Go1.6 中默认启用。在这个机制中，每个项目的根目录都可以有一个 vendor 目录，里面存放了该项目的 Go 依赖包。

在编译 Go 源码时，Go 优先从项目根目录的 vendor 目录查找依赖；如果没有找到，再去 GOPATH 下的 vendor 目录下找；如果还没有找到，就去 GOPATH 下找。这种方式解决了多 GOPATH 的问题，但是随着项目依赖的增多，vendor 目录会越来越大，造成整个项目仓库越来越大

在 vendor 机制下，一个中型项目的 vendor 目录有几百 M 的大小一点也不奇怪。

## Go1.9 版本：Dep
Golang 依赖管理工具混乱的局面最终由官方来终结了：Golang 官方接纳了由社区组织合作开发的 Dep，作为 official experiment。在相当长的一段时间里，Dep 作为标准，成为了事实上的官方包管理工具。

## Go1.11 版本之后：Go Modules
Go1.11 版本推出了 Go Modules 机制，Go Modules 基于 vgo 演变而来，是 Golang 官方的包管理工具。在 Go1.13 版本，Go 语言将 Go Modules 设置为默认的 Go 管理工具；在 Go1.14 版本，Go 语言官方正式推荐在生产环境使用 Go Modules，并且鼓励所有用户从其他的依赖管理工具迁移过来。

## 包（package）和模块（module）
Go 程序被组织到 Go 包中，Go 包是同一目录中一起编译的 Go 源文件的集合。在一个源文件中定义的函数、类型、变量和常量，对于同一包中的所有其他源文件可见。模块是存储在文件树中的 Go 包的集合，并且文件树根目录有 go.mod 文件。go.mod 文件定义了模块的名称及其依赖包，通过导入路径和版本描述一个依赖。

### Go 中有 4 种类型的包：
1. Go 标准包：在 Go 源码目录下，随 Go 一起发布的包
2. 第三方包：第三方提供的包，比如来自于 http://github.com 的包
3. 匿名包：只导入而不使用的包。通常情况下，我们只是想使用导入包产生的副作用，即引用包级别的变量、常量、结构体、接口等，以及执行导入包的init()函数
4. 内部包：项目内部的包，位于项目目录下


## Go Modules 命令
Go Modules 的管理命令为 `go mod`，`go mod` 有很多子命令，你可以通过go help mod来获取所有的命令。
下面我来具体介绍下这些命令:
+ download：下载 go.mod 文件中记录的所有依赖包。
+ edit：编辑 go.mod 文件。
+ graph：查看现有的依赖结构。
+ init：把当前目录初始化为一个新模块。
+ tidy：添加丢失的模块，并移除无用的模块。默认情况下，Go 不会移除 go.mod 文件中的无用依赖。当依赖包不再使用了，可以使用go mod tidy命令来清除它
+ vendor：将所有依赖包存到当前目录下的 vendor 目录下。
+ verify：检查当前模块的依赖是否已经存储在本地下载的源代码缓存中，以及检查下载后是否有修改。
+ why：查看为什么需要依赖某模块。

## Go Modules 开关
通过环境变量 `GO111MODULE` 来打开或者关闭
GO111MODULE 有 3 个值：
+ auto：在 Go1.14 版本中是默认值，在$GOPATH/src下，且没有包含 go.mod 时则关闭 Go Modules，其他情况下都开启 Go Modules。
+ on：启用 Go Modules，Go1.14 版本推荐打开，未来版本会设为默认值。
+ off：关闭 Go Modules，不推荐。

如果要打开 Go Modules，建议直接设置export GO111MODULE=on。

## go.mod 和 go.sum 介绍
go.mod 文件是 Go Modules 的核心文件。
下面是一个 go.mod 文件示例：
```go
module github.com/marmotedu/iam

go 1.17

require (
  github.com/AlekSi/pointer v1.1.0
  github.com/appleboy/gin-jwt/v2 v2.6.3
  github.com/asaskevich/govalidator v0.0.0-20200428143746-21a406dcc535
  github.com/gin-gonic/gin v1.6.3
  github.com/golangci/golangci-lint v1.30.0 // indirect
  github.com/google/uuid v1.0.0
    github.com/blang/semver v3.5.0+incompatible
    golang.org/x/text v0.3.2
)

replace (
    github.com/gin-gonic/gin => /home/colin/gin
    golang.org/x/text v0.3.2 => github.com/golang/text v0.3.2
)

exclude (
    github.com/google/uuid v1.1.0
)
```
### go.mod 语句
o.mod 文件中包含了 4 个语句，分别是 `module`、`require`、`replace` 和 `exclude`。
+ module：用来定义当前项目的模块路径。
+ go：用来设置预期的 Go 版本，目前只是起标识作用。
+ require：用来设置一个 特定 的模块版本，格式为<导入包路径> <版本> [// indirect]。
+ exclude：用来从使用中排除一个特定的模块版本，如果我们知道模块的某个版本有严重的问题，就可以使用 exclude 将该版本排除掉。
+ replace：用来将一个模块版本替换为另外一个模块版本。格式为 `$module => $newmodule` ，$newmodule可以是本地磁盘的'相对路径'，例如 `http://github.com/gin-gonic/gin => ./gin`。也可以是本地磁盘的'绝对路径'，例如 `http://github.com/gin-gonic/gin => /home/lk/gin`。还可以是'网络路径'，例如 `http://golang.org/x/text v0.3.2 => http://github.com/golang/text v0.3.2`。

>PS:replace 之后 要 go mod tidy 重新整理依赖包

## go.mod 版本号
go.mod 文件中有很多版本号格式
+ 如果模块具有符合语义化版本格式的 tag，会直接展示 tag 的值
+ 除了 v0 和 v1 外，主版本号必须显试地出现在模块路径的尾部
+ 对于没有 tag 的模块，Go 命令会选择 master 分支上最新的 commit

### go.mod 文件修改方法
要修改 go.mod 文件，我们可以采用下面这几种方法：
1. 手动编辑 go.mod 文件，编辑之后可以执行 <font color='lightgreen'>go mod edit -fmt</font> 格式化 go.mod 文件
2. 执行 go mod 子命令修改,如下：
```shell
go mod edit -fmt  # go.mod 格式化
go mod edit -require=golang.org/x/text@v0.3.3  # 添加一个依赖
go mod edit -droprequire=golang.org/x/text # require的反向操作，移除一个依赖
go mod edit -replace=github.com/gin-gonic/gin=/home/colin/gin # 替换模块版本
go mod edit -dropreplace=github.com/gin-gonic/gin # replace的反向操作
go mod edit -exclude=golang.org/x/text@v0.3.1 # 排除一个特定的模块版本
go mod edit -dropexclude=golang.org/x/text@v0.3.1 # exclude的反向操作
```

## go.sum介绍
接下来从go.sum 文件内容、go.sum 文件生成、校验三个方面来介绍 go.sum。

### go.sum 文件内容
go.sum 文件中，每行记录由模块名、版本、哈希算法和哈希值组成
正常情况下，每个依赖包会包含两条记录，分别是依赖包所有文件的哈希值和该依赖包 go.mod 的哈希值，例如：
```shell
github.com/appleboy/gin-jwt/v2 v2.6.4 h1:4YlMh3AjCFnuIRiL27b7T0RRUI=
github.com/appleboy/gin-jwt/v2 v2.6.4/go.mod h1:CZpq1cRw+kqi0+yD2CwVs=
```

### go.sum 文件生成
在 Go Modules 开启时，如果我们的项目需要引入一个新的包，通常会执行go get命令，例如：
```shell
$ go get rsc.io/quote
```
当执行go get http://rsc.io/quote命令后，go get命令会先将依赖包下载到$GOPATH/pkg/mod/cache/download，下载的依赖包文件名格式为$version.zip，例如v1.5.2.zip。

下载完成之后，go get会对该 zip 包做哈希运算，并将结果存在$version.ziphash文件中，例如v1.5.2.ziphash。如果在项目根目录下执行go get命令，则go get会同时更新 go.mod 和 go.sum 文件。

### 校验
在我们执行构建时，go 命令会从本地缓存中查找所有的依赖包，并计算这些依赖包的哈希值，然后与 go.sum 中记录的哈希值进行对比。如果哈希值不一致，则校验失败，停止构建。

校验失败可能是因为本地指定版本的依赖包被修改过，也可能是 go.sum 中记录的哈希值是错误的。但是 Go 命令倾向于相信依赖包被修改过，因为当我们在 go get 依赖包时，包的哈希值会经过校验和数据库（checksum database）进行校验，校验通过才会被加入到 go.sum 文件中。也就是说，go.sum 文件中记录的哈希值是可信的。



