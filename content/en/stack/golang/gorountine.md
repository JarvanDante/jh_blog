---
author: "jarvan"
title: "协程池的设计和原理"
date: 2022-11-03 00:00:01
description: "协程池是一种用于管理和复用协程的机制，它可以在并发编程中提供更好的性能和资源利用率"
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
## 协程池
协程池是一种用于管理和复用协程的机制，它可以在并发编程中提供更好的性能和资源利用率。
在Go语言中，协程池可以通过使用 `goroutine` 和 `channel` 来实现。
## 原理
1. 创建一个固定大小的协程池，池中包含多个协程（goroutine）。
2. 当需要执行一个任务时，从协程池中获取一个空闲的协程。
3. 将任务发送到该协程的输入通道（channel）中。
4. 协程从输入通道中接收任务，并执行任务。
5. 执行完任务后，将协程标记为空闲状态，并将自己重新放回协程池中。

通过使用协程池，可以减少协程的创建和销毁开销，提高系统的并发能力和资源利用率。

## 协程池实现
![/images/docImages/pool.png](/images/docImages/pool.png)
以下为协程池的实现：
```golang
package main

import (
	"fmt"
	"time"
)

//------有关Task角色的功能------
//定义一个任务类型Task
type Task struct {
	f func() error //一个Task里面一个具体的任务
}

//创建一个Task任务
func NewTask(arg_f func() error) *Task {
	t := Task{
		f: arg_f,
	}
	return &t
}

//Task也需要一个执行业务的方法
func (t *Task) Execute() {
	t.f() //调用任务中已经绑定的业务方法
}

//------有关Pool角色的功能------
//定义一个Pool协程池的类型
type Pool struct {
	//对外的Task入口 EntryChannel
	EntryChannel chan *Task

	//对内部的Task队列 JobsChannel
	JobsChannel chan *Task

	//协程池中最大的worker的数量
	worker_num int
}

//创建Pool的函数
func NewPool(cap int) *Pool {
	//创建一个Pool
	p := Pool{
		EntryChannel: make(chan *Task),
		JobsChannel:  make(chan *Task),
		worker_num:   cap,
	}
	//返回这个Pool
	return &p
}

//协程池创建一个worker，并且让这个worker去工作
func (p *Pool) worker(work_id int) {
	//一个worker具体的工作

	//1永久的从JobsChannel去取任务
	for task := range p.JobsChannel {
		//task 就是当前worker从JobsChannel中拿到的任务
		//2 一旦取到任务，执行这个任务
		task.Execute()
		fmt.Println("worker ID", work_id, " 执行完了一个任务")
	}

}

//让协程池，开始真正的工作，协程池一个启动方法
func (p *Pool) run() {
	//1 根据worker_num 来创建worker去工作
	for i := 0; i < p.worker_num; i++ {
		//每个worker都应该是一个goroutine
		go p.worker(i)
	}
	//2 从EntryChannel 中去取任务，将取到的任务，发送给JobsChannel
	for task := range p.EntryChannel {
		//一旦有task 读到
		p.JobsChannel <- task
	}
}

//主函数，执行协程池的工作
func main() {
	//1创建一些任务
	t := NewTask(func() error {
		fmt.Println(time.Now())
		return nil
	})
	//2创建一个Pool协程池
	p := NewPool(4)
	task_num := 0
	//3将这些任务交给协程池
	go func() {
		for {
			p.EntryChannel <- t
			task_num++
			fmt.Println("一共执行数：--", task_num)
		}
	}()

	//4启动pool，让pool开始工作
	p.run()
}

```

以下是一个简单的示例代码，演示了如何使用协程池来执行任务：
```golang
package main

import (
	"fmt"
	"sync"
)

type Task struct {
	ID   int
	Data string
}

func worker(id int, tasks <-chan Task, wg *sync.WaitGroup) {
	defer wg.Done()

	for task := range tasks {
		fmt.Printf("Worker %d processing task %d with data: %s\n", id, task.ID, task.Data)
		// 执行任务的逻辑代码
	}
}

func main() {
	poolSize := 5
	taskCount := 10

	// 创建任务通道和等待组
	tasks := make(chan Task, taskCount)
	var wg sync.WaitGroup

	// 创建协程池
	for i := 1; i <= poolSize; i++ {
		wg.Add(1)
		go worker(i, tasks, &wg)
	}

	// 添加任务到通道中
	for i := 1; i <= taskCount; i++ {
		tasks <- Task{ID: i, Data: fmt.Sprintf("data-%d", i)}
	}
	close(tasks)

	// 等待所有任务完成
	wg.Wait()
}

```
