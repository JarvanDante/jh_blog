---
author: "jarvan"
title: "goframe目录"
date: 2021-02-01 01:01:00
description: "go语言起源、安装运行环境、编辑器、集成等"
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

## 一、工程目录结构
GoFrame业务项目基本目录结构如下（以Single Repo为例）：
```shell
/
├── api
├── hack
├── internal
│   ├── cmd
│   ├── consts
│   ├── controller
│   ├── dao
│   ├── logic
│   ├── model
│   |   ├── do
│   │   └── entity
│   └── service
├── manifest
├── resource
├── utility
├── go.mod
└── main.go 
```
| 目录/文件名称       | 说明         | 描述                                                        |
|:--------------|:-----------|:----------------------------------------------------------|
| api           | 对外接口       | 对外提供服务的输入/输出数据结构定义。考虑到版本管理需要，往往以api/v1...存在。              |
| hack          | 工具脚本       | 存放项目开发工具、脚本等内容。例如，CLI工具的配置，各种shell/bat脚本等文件。              |
| internal      | 内部逻辑       | 业务逻辑存放目录。通过Golang internal特性对外部隐藏可见性。                     |
| - cmd         | 入口指令	      | 命令行管理目录。可以管理维护多个命令行。                                      |
| - consts      | 常量定义       | 项目所有常量定义。                                                 |
| - controller  | 接口处理       | 接收/解析用户输入参数的入口/接口层。                                       |
| - dao         | 数据访问	      | 数据访问对象，这是一层抽象对象，用于和底层数据库交互，仅包含最基础的 CURD 方法                |
| - logic       | 业务封装	      | 业务逻辑封装管理，特定的业务逻辑实现和封装。往往是项目中最复杂的部分。                       |
| - model	      | 结构模型	      | 数据结构管理模块，管理数据实体对象，以及输入与输出数据结构定义。                          |
| - - do        | 领域对象	      | 用于dao数据操作中业务模型与实例模型转换，由工具维护，用户不能修改。                       |
| - - entity    | 数据模型	      | 数据模型是模型与数据集合的一对一关系，由工具维护，用户不能修改。                          |
| - service     | 业务接口	      | 用于业务模块解耦的接口定义层。具体的接口实现在logic中进行注入。                        |
| manifest	     | 交付清单	      | 包含程序编译、部署、运行、配置的文件。常见内容如下：                                |
|   - config     | 配置管理	      | 配置文件存放目录。                                                 |
|   - docker     | 镜像文件       | Docker镜像相关依赖文件，脚本文件等等。                                    |
|   - deploy     | 部署文件	   | 部署相关的文件。默认提供了Kubernetes集群化部署的Yaml模板，通过kustomize管理。        |
| resource	    |   静态资源	  | 静态资源文件。这些文件往往可以通过 资源打包/镜像编译 的形式注入到发布文件中。                  |
|  go.mod	   |  依赖管理	  | 使用Go Module包管理的依赖描述文件。                                    |
|   main.go	  |  入口文件	  |    程序入口文件。|

+ 对外接口
  对外接口包含两部分：接口定义（api）+接口实现（controller）。
  服务接口的职责类似于三层架构设计中的UI表示层，负责接收并响应客户端的输入与输出，包括对输入参数的过滤、转换、校验，对输出数据结构的维护，并调用 service 实现业务逻辑处理。
+ 接口定义 - api
  api包用于与客户端约定的数据结构输入输出定义，往往与具体的业务场景强绑定。


+ 接口实现 - controller
  controller用于接收api的输入，调用内部的一个或多个service包实现业务场景，组织service的结果构造为api的输出数据结构。

+ 业务实现
  业务实现包含两部分：业务接口（service）+业务封装（logic）。

业务实现的职责类似于三层架构设计中的BLL业务逻辑层，负责具体业务逻辑的实现以及封装。
+ 业务接口 - service
  service包用于解耦业务模块之间的调用。业务模块之间往往不会直接调用对应的业务模块资源来实现业务逻辑，而是通过调用service接口。service层只有接口定义，具体的接口实现注入在各个业务模块中。
+ 业务封装 - logic
  logic包负责具体业务逻辑的实现以及封装。项目中各个层级代码不会直接调用logic层的业务模块，而是通过service接口层来调用。

+ 结构模型
  model包的职责类似于三层架构中的Model模型定义层。模型定义代码层中仅包含全局公开的数据结构定义，往往不包含方法定义。

这里需要注意的是，这里的model不仅负责维护数据实体对象（entity）结构定义，也包括所有的输入/输出数据结构定义，被api/dao/service共同引用。这样做的好处除了可以统一管理公开的数据结构定义，也可以充分对同一业务领域的数据结构进行复用，减少代码冗余。
+ 数据模型 - entity
  与数据集合绑定的程序数据结构定义，通常和数据表一一对应。

+ 业务模型 - model
  与业务相关的通用数据结构定义，其中包含大部分的方法输入输出定义。

+ 数据访问 - dao
  dao包的职责类似于三层架构中的DAL数据访问层，数据访问层负责所有的数据访问收口。


## 二、请求分层流转
+ cmd
  cmd层负责引导程序启动，显著的工作是初始化逻辑、注册路由对象、启动server监听、阻塞运行程序直至server退出。
+ api
  上层server服务接收客户端请求，转换为api中定义的Req接收对象、执行请求参数到Req对象属性的类型转换、执行Req对象中绑定的基础校验并转交Req请求对象给controller层。
+ controller
  controller层负责接收Req请求对象后做一些业务逻辑校验，随后调用一个或多个service实现业务逻辑，将执行结构封装为约定的Res数据结构对象返回。
+ model
  model层中管理了所有的业务模型，service资源的Input/Output输入输出数据结构都由model层来维护。
+ service
  service是接口层，用于解耦业务模块，service没有具体的业务逻辑实现，具体的业务实现是依靠logic层注入的。
+ logic
  logic层的业务逻辑需要通过调用dao来实现数据的操作，调用dao时需要传递do数据结构对象，用于传递查询条件、输入数据。dao执行完毕后通过Entity数据模型将数据结果返回给service层。
+ dao
  dao层通过框架的ORM抽象层组件与底层真实的数据库交互。
  ![/images/docImages/goframe1.png](/images/docImages/goframe1.png)