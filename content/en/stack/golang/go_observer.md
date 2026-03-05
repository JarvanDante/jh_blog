---
author: "karson"
title: "Golang的观察者模式"
date: 2021-01-01 00:00:10
description: "定义了对象之间的一对多依赖关系，使得当一个对象改变状态时，其所有依赖对象都会收到通知并自动更新"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags: 
- golang
categories:
- golang



---
定义了对象之间的一对多依赖关系，使得当一个对象改变状态时，其所有依赖对象都会收到通知并自动更新

>*个人理解说法：
> 步骤1 定义一个观察者的接口，有一个方法update(string)
> 步骤2 定义一个主题结构体，包含多个观察者的切片 和 状态 字段，主要结构体有自己的三个方法，Attach(追加观察者)、SetState(设置状态)、Notify(通知所有观察者修改状态) 。
> 步骤3 实例化可以实现观察者接口的观察者实例 和 对应的 update(string)方法
> 步骤4 执行：实例化 观察者1 和 观察者2，初始化主题结构体，把观察者1，2都attache到切片中，之后使用主题的SetState设置状态，所有观察者都修改

如下：
```go
package main

import "fmt"

type Observer interface {
	Update(string string)
}

type Subject struct {
	observers []Observer
	state     string
}

func (s *Subject) Attach(observer Observer) {
	s.observers = append(s.observers, observer)
}

func (s *Subject) SetState(state string) {
	s.state = state
	s.Notify()
}

func (s *Subject) Notify() {
	for _, observer := range s.observers {
		observer.Update(s.state)
	}
}

type ConcreteObserver struct {
	name string
}

func (co *ConcreteObserver) Update(state string) {
	fmt.Printf("Observer %s received the state: %s\n", co.name, state)
}

func main() {
	subject := &Subject{}
	observer1 := &ConcreteObserver{name: "Observer 1"}
	observer2 := &ConcreteObserver{name: "Observer 2"}

	subject.Attach(observer1)
	subject.Attach(observer2)

	subject.SetState("new state")
}
```

运行如下：
```shell
$go run main.go
Observer Observer 1 received the state: new state
Observer Observer 2 received the state: new state
```