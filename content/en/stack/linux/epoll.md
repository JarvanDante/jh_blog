---
author: "jarvan"
title: "epoll模型的原理和工作过程"
date: 2022-11-02 00:00:01
description: "epoll是Linux操作系统中的一种事件通知机制，通过该机制可以同时监控多个文件描述符上的事件"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags: 
- linux
- categories:

---


## 常见的网络I/O模型有哪些
------------------------------------------------
###  1.阻塞式I/O（Blocking I/O）
应用程序通过调用系统提供的I/O函数进行数据读写时，会阻塞等待直到数据传输完成。这种模型适用于简单的应用场景，但在处理多个并发连接时效率较低。

###  2.非阻塞式I/O（Non-blocking I/O）
应用程序使用非阻塞的方式调用I/O函数，可以立即返回，不会等待数据的传输完成。通过不断轮询判断是否有数据可读或可写，实现了并发处理。但是，轮询会消耗大量的CPU资源。

###  3.I/O多路复用（I/O Multiplexing）
使用操作系统提供的select、poll或epoll等函数，将多个文件描述符注册到一个集合中，通过调用这些函数等待数据的就绪状态。当有数据到达时，返回就绪的文件描述符，应用程序可以进行相应的处理。相较于非阻塞式I/O，I/O多路复用减少了无效轮询的消耗。
#### 说明:文件描述符（fd）
fd就是(file discriptor)为文件描述符。socket起源于unix，unix中把所有的资源都看作是文件，如网卡、打印机等。
简单点说也就是` int fd = socket(AF_INFT,SOCK_STREAM,0); `函数socket()返回的是这个描述符。

####  epoll的实现
在Linux系统中，epoll是一种高性能的I/O事件通知机制，用于处理大量的并发连接。epoll提供了一组系统调用接口，用于注册、管理和等待I/O事件。
epoll的接口主要包括以下几个函数：

1. epoll_create：创建一个epoll实例，返回一个文件描述符，用于后续的操作。

2. epoll_ctl：向epoll实例中注册或删除文件描述符，以及设置关注的事件类型。

   + EPOLL_CTL_ADD：将文件描述符添加到epoll实例中。
   + EPOLL_CTL_MOD：修改已注册的文件描述符的关注事件类型。
   + EPOLL_CTL_DEL：从epoll实例中删除已注册的文件描述符。

3. epoll_wait：等待事件的发生，返回就绪的文件描述符和事件信息。

   + events参数：用于接收就绪的事件信息的数组。
   + maxevents参数：指定events数组的大小，表示最多等待多少个事件。
   + timeout参数：指定等待的超时时间，可以设为-1表示无限等待。

```shell
events常用参数：
EPOLLIN：表示文件描述符可读，即有数据可以从文件描述符中读取。
EPOLLOUT：可写，即可以向文件描述符中写入数据。
EPOLLERR：发生错误，需要进行错误处理。
EPOLLPRI：有紧急数据可读，需要进行处理。
EPOLLHUP：挂起，即文件描述符被挂起或关闭，需要进行处理。
EPOLLET：设置为边缘触发模式，即只有状态改变时才会触发事件。
EPOLLONESHOT：表示只监听一次事件，当事件触发后，需要重新设置监听。
这些宏可以在使用epoll进行事件监听时，通过设置epoll_event结构体的events字段来指定所关注的事件类型。
```
以下演示如何使用epoll的接口来实现事件驱动的网络服务器：
```golang
package main

import (
	"fmt"
	"log"
	"net"
	"os"
	"syscall"
)

func main() {
	// 创建epoll实例
	epollFd, err := syscall.EpollCreate1(0)
	if err != nil {
		log.Fatal(err)
	}

	// 创建监听套接字
	listener, err := net.Listen("tcp", "127.0.0.1:8080")
	if err != nil {
		log.Fatal(err)
	}
	defer listener.Close()

	// 将监听套接字添加到epoll实例中
	event := syscall.EpollEvent{
		Events: syscall.EPOLLIN,
		Fd:     int32(listener.(*net.TCPListener).Fd()),
	}
	err = syscall.EpollCtl(epollFd, syscall.EPOLL_CTL_ADD, int(listener.(*net.TCPListener).Fd()), &event)
	if err != nil {
		log.Fatal(err)
	}

	events := make([]syscall.EpollEvent, 10)

	for {
		// 等待事件的发生
		n, err := syscall.EpollWait(epollFd, events, -1)
		if err != nil {
			log.Fatal(err)
		}

		// 处理就绪的事件
		for i := 0; i < n; i++ {
			if int(events[i].Fd) == int(listener.(*net.TCPListener).Fd()) {
				// 接受新的连接
				conn, err := listener.Accept()
				if err != nil {
					log.Println(err)
					continue
				}

				// 处理连接
				go handleConnection(conn)
			}
		}
	}
}

func handleConnection(conn net.Conn) {
	defer conn.Close()

	// 处理连接的业务逻辑
	fmt.Fprintf(conn, "Hello, client!\n")
}
```
这只是一个简化的示例，实际使用epoll时还需要处理错误、设置非阻塞模式、处理其他事件类型等。详细的epoll接口使用可以参考相关的文档和资料。

#### ET LT：
1. 水平触发（LT）是指当socket接收缓冲区中有数据可读时，读事件会持续触发；当socket发送缓冲区未满时，可以继续写入数据，写事件也会持续触发。

2. 边缘触发（ET）是指当socket接收缓冲区发生变化时，即空的接收缓冲区刚接收到数据时，会触发读事件；当socket发送缓冲区状态发生变化时，即满的缓冲区刚空出空间时，会触发写事件。

##### LT的处理过程：
LT的处理过程：
accept一个连接，添加到epoll中监听EPOLLIN事件。

当EPOLLIN事件到达时，read fd中的数据并处理，

当需要写出数据时，把数据write到fd中；如果数据较大，无法一次性写出，那么在epoll中监听EPOLLOUT事件。

当EPOLLOUT事件到达时，继续把数据write到fd中 ；如果数据写出完毕，那么在epoll中关闭EPOLLOUT事件。

ET的处理过程如下：

接受一个连接，并将其添加到epoll中，监听EPOLLIN|EPOLLOUT事件。

当EPOLLIN事件到达时，从文件描述符（fd）中读取数据并进行处理。需要一直读取，直到返回EAGAIN为止。

当需要写出数据时，将数据写入fd中，直到数据全部写完或者write返回EAGAIN。

当EPOLLOUT事件到达时，继续将数据写入fd中，直到数据全部写完或者write返回。

ET模式accept存在的问题：
多个连接同时到达时，服务器TCP会瞬间积累多个就绪连接。由于是边缘触发模式，epoll只会通知一次，而accept只处理一个连接，这导致TCP就绪队列中剩下的连接都得不到处理。为了解决这个问题，可以在while循环中使用accept调用，处理完就绪队列中的所有连接后再退出循环。要判断是否处理完所有连接，可以通过accept返回-1并且将error设置为errno的值EAGAIN来判断。

LT==>只要event为EPOLLIN时，就会不断调用回调函数。

ET==>只有当从EPOLLOUT变化为EPOLLIN时，才会触发。
###  4.信号驱动式I/O（Signal-driven I/O）
应用程序将文件描述符设置为非阻塞模式，并使用异步I/O函数（如aio_read、aio_write）进行数据的读写操作。当数据就绪时，操作系统会发送一个信号给应用程序，应用程序通过信号处理函数来处理数据。这种模型较少使用。

###  5.异步I/O（Asynchronous I/O）
应用程序发起I/O操作后，通过回调函数或事件通知的方式来处理数据读写。操作系统会在数据就绪时通知应用程序，应用程序继续处理其他任务，当操作系统完成数据传输后，再执行回调函数来处理数据。这种模型相较于其他模型有更好的性能表现，但编程复杂度较高。


