---
author: "jarvan"
title: "goroutine并发实现"
date: 2022-11-02 00:00:01
description: "epoll是Linux操作系统中的一种事件通知机制，通过该机制可以同时监控多个文件描述符上的事件"
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


## 常见并发模型
### 进程&线程(apache)
C10K:C10K problem是指如何让服务器能够支持10k并发，当然你可以买昂贵的服务器，但是还有更便宜的办法。

### 异步非阻塞(nginx/libevent/nodejs)
复杂度高
### 协程(golang/erlang/lua)
+ 程序并发执行（goroutine）
  goroutines（程序并发执行）
```golang
foo()//函数
go foo() //执行函数foo
bar() //不用等待foo返回
```
+ 多个goroutine间的数据同步和通信（channels）
channels（多个goroutine间的数据通信与同步）
```golang
c:=make(chan string) //创建一个channel
go func(){
   time.Sleep(1*time.Second)
   c<-"message from closure" //发送数据到channel中
}()
msg:=<-c //阻塞直到接收到数据 
```
+ 多个channel选择数据读取或写入（select）
select(从多个channel中读取或写入数据)
```golang
select{
case v := <-c1:
   fmt.Println("channel 1 sends",v)
case v := <-c2:
   fmt.Println("channel 2 sends",v)
default://可选
   fmt.Println("neither channel was ready")
} 
```

## golang中的面向对象
### 1. 封装
实现：
```golang
type Foo struct{
   baz string
}
func (f *Foo) echo (){
   fmt.Println(f.baz)
}
func main(){
   f :=Foo{baz:"hello , struct"}
   f.echo()
}
```
### 2. 继承
```golang
type Foo struct{
   baz string
} 
type Bar struct{
   Foo
}
func (f *Foo) echo (){
   fmt.Println(f.baz)
}
func main(){
   b := Bar{Foo{baz:"hello, struct"}}
   b.echo()
}
```
### 3. 多态
```golang
type Foo interface{
   qux()
} 
type Bar struct{}
type Baz struct{}
func (b Bar) qux(){}
func (b Baz) qux(){}
func main(){
   var f Foo
   f = Bar{}
   f = Baz{}
   fmt.Println(f)
}
```

协和例子：
```golang
package main

import (
	"fmt"
	"strings"
	"time"
)

type LogProcess struct {
	rc          chan string
	wc          chan string
	path        string //读取文件的路径
	influxDBDsn string //data source
}

func (l *LogProcess) ReadFromFile() {
	//读取模块
	line := "message"
	l.rc <- line
}

func (l *LogProcess) Process() {
	//解析模块
	data := <-l.rc
	l.wc <- strings.ToUpper(data)

}

func (l *LogProcess) WriteToInfluxDB() {
	//写入模块
	fmt.Println(<-l.wc)
}

func main() {
	lp := &LogProcess{
		rc:          make(chan string),
		wc:          make(chan string),
		path:        "/tmp/access.log",
		influxDBDsn: "user&pwd",
	}
	go lp.ReadFromFile()
	go lp.Process()
	go lp.WriteToInfluxDB()
	
	time.Sleep(1 * time.Second)
}
```

读取、写入部分模块用接口独立出来
```golang
package main

import (
	"fmt"
	"strings"
	"time"
)

type Reader interface {
	Read(rc chan string)
}

type Writer interface {
	Write(wc chan string)
}
type ReadFromFile struct {
	path string //读取文件的路径
}

func (r *ReadFromFile) Read(rc chan string) {
	//读取模块
	line := "message"
	rc <- line
}

type WriteToInfluxDB struct {
	influxDBDsn string //data source
}

func (w *WriteToInfluxDB) Write(wc chan string) {
	//写入模块
	fmt.Println(<-wc)
}

type LogProcess struct {
	rc    chan string
	wc    chan string
	read  Reader
	write Writer
}

func (l *LogProcess) Process() {
	//解析模块
	data := <-l.rc
	l.wc <- strings.ToUpper(data)

}

func main() {
	r := &ReadFromFile{
		path: "/tmp/access.log",
	}
	w := &WriteToInfluxDB{
		influxDBDsn: "user&pwd",
	}
	lp := &LogProcess{
		rc:    make(chan string),
		wc:    make(chan string),
		read:  r,
		write: w,
	}
	go lp.read.Read(lp.rc)
	go lp.Process()
	go lp.write.Write(lp.wc)

	time.Sleep(1 * time.Second)
}

```
实现简单日志读取：
```golang
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
	"strings"
	"time"
)

type Reader interface {
	Read(rc chan []byte)
}

type Writer interface {
	Write(wc chan string)
}
type ReadFromFile struct {
	path string //读取文件的路径
}

func (r *ReadFromFile) Read(rc chan []byte) {
	//读取模块
	//打开文件
	f, err := os.Open(r.path)
	if err != nil {
		panic(err)
	}
	//从文件末尾读取文件内容
	f.Seek(0, 2)
	rd := bufio.NewReader(f)
	for {
		line, err := rd.ReadBytes('\n')
		if err == io.EOF {
			time.Sleep(500 * time.Millisecond)
			continue
		} else if err != nil {
			panic(err)
		}
		//line := "message"
		rc <- line[:len(line)-1]
	}

}

type WriteToInfluxDB struct {
	influxDBDsn string //data source
}

func (w *WriteToInfluxDB) Write(wc chan string) {
	//写入模块
	for v := range wc {
		fmt.Println(v)
	}

}

type LogProcess struct {
	rc    chan []byte
	wc    chan string
	read  Reader
	write Writer
}

func (l *LogProcess) Process() {
	//解析模块
	for v := range l.rc {
		l.wc <- strings.ToUpper(string(v))
	}

}

func main() {
	r := &ReadFromFile{
		path: "./access.log",
	}
	w := &WriteToInfluxDB{
		influxDBDsn: "user&pwd",
	}
	lp := &LogProcess{
		rc:    make(chan []byte),
		wc:    make(chan string),
		read:  r,
		write: w,
	}
	go lp.read.Read(lp.rc)
	go lp.Process()
	go lp.write.Write(lp.wc)

	time.Sleep(30 * time.Second)
}

```

运行：
```shell
go run log_process.go
# 之后输入日志内容
echo 123123123>>access.log
```