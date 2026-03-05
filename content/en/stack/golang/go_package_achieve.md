---
author: "karson"
title: "Golang的封装和实现"
date: 2021-01-01 00:00:5
description: "在Go语言中封装就是把抽象出来的字段和对字段的操作封装在一起，数据被保护在内部，程序的其它包只能通过被授权的方法，才能对字段进行操作"
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
## 封装
在Go语言中封装 就是 把 <font color='cyan'>**抽象出来的字段**</font> 和 <font color='cyan'>**对字段的操作**</font> 封装在一起，数据被保护在内部，程序的其它包只能通过被授权的方法，才能对字段进行操作

### 封装的好处：
+ 隐藏实现细节；
+ 可以对数据进行验证，保证数据安全合理
### 如何实现封装：
+ 对结构体中的属性进行封装；
+ 通过方法，包，实现封装

### 封装的实现步骤：
1. 将<font color='cyan'>**结构体、字段**</font> 的首字母 <font color='cyan'>**小写**</font>
2. 给结构体所在的包提供一个 <font color='cyan'>**工厂模式的函数 （NewPerson构造函数）**</font> ，首字母大写，类似一个构造函数
3. 提供一个首字母大写的 <font color='cyan'>**Set 方法**</font>（类似其它语言的 public），用于对属性 **判断**并**赋值**
4. 提供一个首字母大写的 <font color='cyan'>**Get 方法**</font>（类似其它语言的 public），用于获取属性的 **值**

示例：
对于员工，不能随便查看年龄，工资等隐私，并对输入的年龄进行合理的验证。
代码结构如下：
```shell
company
├── main
│   └── main.go
└── model
    └── person.go
```

person.go 中的代码如下所示：
```go
package model

import "fmt"

type person struct {
	Name string
	age  int //其它包不能访问..
}

// 写一个工厂模式的函数，相当于构造函数
func NewPerson(name string) *person {
	return &person{
		Name: name,
	}
}

// 为了访问age和sal我们编写一个SetXxx的方法和GetXxx的方法
func (p *person) SetAge(age int) {
	if age > 0 && age < 150 {
		p.age = age
	} else {
		fmt.Println("年龄范围不正确..")
	}
}
func (p *person) GetAge() int {
	return p.age

}
```
main.go 中的代码如下所示：
```go
package main

import (
	"fmt"
	"kg_info_dashboard_api/company/model"
)

func main() {
	p := model.NewPerson("jay")
	p.SetAge(18)
	fmt.Println(p)
	fmt.Println(p.Name, "age=", p.GetAge())
}
```
执行结果如下：
```shell
$ go run main.go 
&{jay 18}
jay age= 18
```


