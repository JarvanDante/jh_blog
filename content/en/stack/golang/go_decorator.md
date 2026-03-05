---
author: "jarvan"
title: "Golang的装饰器模式"
date: 2021-01-01 00:00:09
description: "装饰器模式可通过在接口中封装其它接口并添加行为来实现"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags: 
- golang
categories:
- golang



---
装饰器模式可通过在接口中封装其它接口并添加行为来实现
如下：
```go
package main

import "fmt"

type Component interface {
    Operation() string
}

type ConcreteComponent struct{}

func (c *ConcreteComponent) Operation() string {
    return "Concrete Component"
}

type Decorator struct {
    component Component
}

func (d *Decorator) Operation() string {
    return "Decorator " + d.component.Operation()
}

func main() {
    component := &ConcreteComponent{}
    decorator := &Decorator{component: component}

    fmt.Println(component.Operation())
    fmt.Println(decorator.Operation())
}
```

结果：
```shell
$go run main.go
Concrete Component
Decorator Concrete Component
```