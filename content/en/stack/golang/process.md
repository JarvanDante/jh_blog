---
author: "jarvan"
title: "goroutine概念"
date: 2022-11-03 00:00:01
description: "Goroutine可以理解为一种Go语言的协程（轻量级线程)，是Go支持高并发的基础，属于用户态的线程，由Goruntime管理而不是操作系统。"
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

## 进程
进程(Process)就是程序在操作系统中的 <font color='lightgreen'>一次执行过程</font>， 是系统进行资源分配和调度的基本单位，进程是一个动态概念，是程序在执行过程中分配和管理资源的基本单位，每一个进程都有一个自己的地址空间。
一个进程至少有 5 种基本状态，它们是：初始态，执行态，等待状态，就绪状态，终止状态。
<font color='lightgreen'>通俗的讲 **进程** 就是 一个正在执行的程序</font>。

## 线程
<font color='lightgreen'>线程 是进程的一个执行实例</font>，是程序执行的最小单元，它是比进程更小的能独立运行的基本单位。
一个进程可以创建多个线程，同一个进程中的多个线程可以并发执行，一个程序要运行至少有一个进程。

## 关于并行和并发
### 并发：
并发：多个线程同时竞争一个位置，竞争到的才可以执行，每一个时间段 <font color='lightgreen'>只有一个线程 </font>在执行。
### 并行：
并行：多个线程可以同时执行，每一个时间段，可以有 <font color='lightgreen'>多个线程同时执行 </font>。

**通俗的讲** ： 多线程程序在 <font color='lightgreen'>单核CPU上面运行</font> 就是 **并发**， 多线程程序在 <font color='lightgreen'>多核CPU</font> 上运行就是 **并行**，
如果线程数大于CPU术数，则多线程程序在多个CPU上面运行既有并行又有并发。

## 协程
golang中的主线程：在一个golang程序的主线程上可以起多个协程。golang中多协程可以实现并行或并发。
协程：可以理解为 <font color='lightgreen'>用户级线程</font>，完全由用户自己的程序进行调度的。Golang的一大特色就是
从语言层面原生支持协程，在函数或者方法前面加go关键字就可以创建一个协程。可以说Golang中的协程就是goroutine。

JAVA/C 中开一个线程大概消耗 <font color='lightgreen'>2MB</font> 左右，一个goroutine占用内存非常小，只有 <font color='lightgreen'>2KB</font> 左右。

## 协程执行完毕
### sync.WaitGroup
```golang
var wg sync.WaitGroup

func test(){
    ...
    wg.Done() //协程计数器加 -1
}
func main(){
    wg.add(1) //协程计数器加 1
    go test() 
    ...
    wg.Wait() //等待协程执行完毕...计数器为0
    
}
```

+ 获取当前计算机上面的CUP个数
```golang
cpuNum:=runtime.NumCPU() 
```
+ 可以自己设置使用多个CPU
```golang
runtime.GOMAXPROCS(cpuNum - 1)
```