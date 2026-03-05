---
author: "jarvan"
title: "Serverless"
date: 2022-11-04 00:00:01
description: "serverless狭义上是指Serverless computing 架构 = FaaS 架构 = Trigger(事件驱动)+FaaS(函数即服务)+BaaS(后端即服务，持久化或第三方服务)=FaaS + BaaS，广义上是指服务端免运维，也就是具有 Serverless 特性的云服务"
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

无服务器
不够灵活,难以预料的波动
无需CDN

历史：
serverless=FaaS(Function as a Service)+BaaS(Backend as a Service)

浪费
好处：
1、使用的资源付费
2、服务高可用，弹性伸缩
3、无运维费用
缺点：
冷启动慢
## socket概念
在计算机网络中，Socket是一种<font color='#aed581'>抽象概念</font>，它可以看作是一种端点，用于标识网络中的一个通信连接。一个Socket由 <font color='#aed581'>IP地址</font> 和 <font color='#aed581'>端口号</font> 组成，它可以唯一地标识网络中的一个进程。

Socket编程是一种在网络上进行通信的编程方式，通过使用Socket接口，开发人员可以编写程序来创建、连接、发送和接收数据，从而实现网络通信。Socket编程可以用于实现各种网络应用，如Web服务器、邮件服务器、聊天程序等。

在Socket编程中，通常有两种类型的Socket：

1. 流式Socket（Stream Socket）：也称为TCP Socket，它提供了可靠的、面向连接的通信。流式Socket使用TCP协议，在通信之前需要先建立连接，然后进行数据的传输，确保数据的可靠性和顺序。流式Socket适用于需要可靠传输的应用，如文件传输、HTTP通信等。

2. 数据报式Socket（Datagram Socket）：也称为UDP Socket，它提供了无连接的通信。数据报式Socket使用UDP协议，每个数据包都是独立的，不需要事先建立连接，因此传输速度较快，但不保证数据的可靠性和顺序。数据报式Socket适用于实时性要求较高的应用，如视频流传输、实时游戏等。

通过Socket编程，可以实现不同计算机之间的通信和数据交换，使得网络应用能够实现数据的传输和交互。
## TCP编程
### TCP/IP
TCP/IP即传输控制协议/网络协议，是一种面向连接的、可靠的、基于字节流的传输层通信协议，因为是面向连接的协议，数据像水流一样传输，会存在 <font color="#aed581">黏包</font> 的问题
#### 黏包
黏包（Packet Sticking）是指发送方将多个小数据包粘合在一起发送，或接收方将接收到的数据包粘合在一起，导致数据的边界不清晰，难以正确解析和处理。

##### 原因：

1. 发送方的缓冲区未满：
发送方的缓冲区未满时，可能会将多个小数据包一起发送，导致接收方接收到的数据包粘合在一起。
2. 网络传输过程中的分段和重组：
TCP协议在传输过程中会对数据进行分段和重组，这可能导致接收方接收到的数据包与发送方发送的数据包大小不一致。
3. 接收方的缓冲区未及时读取：
接收方的缓冲区未及时读取已接收的数据，导致后续的数据包与之前的数据包粘合在一起。

##### 解决方案：
1. 消息长度固定：发送方在每个数据包中添加固定长度的消息头，用于标识消息的长度。接收方根据消息头中的长度信息，将接收到的数据包正确地拆分成单个消息进行处理。
2. 消息分隔符：发送方在每个数据包的末尾添加特定的分隔符（如换行符或特殊字符），接收方根据分隔符将接收到的数据包正确地拆分成单个消息进行处理。
3. 消息头中包含消息长度信息：发送方在每个数据包的消息头中添加消息的长度信息，接收方根据消息头中的长度信息，将接收到的数据包正确地拆分成单个消息进行处理。
4. 使用定长消息：发送方将所有的消息都固定为相同的长度，不足部分使用特定字符进行填充。接收方根据固定长度将接收到的数据包正确地拆分成单个消息进行处理。
5. 应用层协议处理：在应用层定义自己的协议，规定消息的格式和边界，确保发送和接收的数据能够正确解析和处理。

### TCP服务端
一个TCP服务端可以同时连接很多个客户端，如世界各地的用户用自己的电脑访问淘宝。因为go语言中创建多个goroutine实现并发非常方便和高效，所以我们可以每建立一次连接就创建一个goroutine去处理。

TCP服务端程序的处理流程：
1. `监听端口`
2. `接收客户端请求建立连接`
3. `创建goroutine处理连接`

使用go语言中的net包实现TCP服务端代码如下：
```golang
package main

import (
	"bufio"
	"fmt"
	"net"
)

func process(conn net.Conn) {
	defer conn.Close() //关闭连接
	for {
		reader := bufio.NewReader(conn)
		var buf [128]byte
		n, err := reader.Read(buf[:]) //读取数据
		if err != nil {
			fmt.Println("read from client failed,err:", err)
			break
		}
		recvStr := string(buf[:n])
		fmt.Println("收到client端发来的数据：", recvStr)
		conn.Write([]byte(recvStr)) //发送数据
	}
}

func main() {
	listen, err := net.Listen("tcp", "127.0.0.1:20000")
	if err != nil {
		fmt.Println("listen failed,err:", err)
		return
	}
	for {
		conn, err := listen.Accept() //建立连接
		if err != nil {
			fmt.Println("accept failed,err:", err)
			continue
		}
		go process(conn) //启动一个goroutine处理连接
	}
}
```

### TCP客户端
一个TCP客户端进行TCP通信的流程：
1. `建立与服务端的连接`
2. `进行数据收发`
3. `关闭连接`

使用go语言的net包实现TCP客户端代码如下：
```golang
package main

import (
	"bufio"
	"fmt"
	"io"
	"net"
	"os"
	"strings"
)

func main() {
	conn, err := net.Dial("tcp", "127.0.0.1:20000")

	if err != nil {
		fmt.Println("err:", err)
		return
	}
	defer conn.Close() //关闭连接
	inputReader := bufio.NewReader(os.Stdin)
	for {
		input, _ := inputReader.ReadString('\n') //读取用户输入
		inputInfo := strings.Trim(input, "\r\n")
		if strings.ToUpper(inputInfo) == "Q" {
			return
		}
		_, err = conn.Write([]byte(inputInfo)) //发送数据
		if err != nil {
			return
		}
		buf := [512]byte{}
		n, err := conn.Read(buf[:])
		if err != nil {
			if err == io.EOF {
				break
			}
			fmt.Println("recv failed, err:", err)
			return
		}
		fmt.Println("接收到服务器发来的数据：" + string(buf[:n]))
	}
}

```

## UDP编程
### UDP协议
UDP协议（用户数据报协议），是OSI模型中的一种无连接的传输层协议，不需要建立连接就能直接进行数据发送和接收，属于不可靠的、没有时序的通信，但是UDP协议的实时性比较好，通常用于视频直播相关领域。

### UDP服务端
使用go语言的net包实现的UDP服务端代码如下：
```golang
package main

import (
	"fmt"
	"net"
)

func main() {
	listen, err := net.ListenUDP("udp", &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("listen failed,err:", err)
		return
	}
	defer listen.Close()
	for {
		var data [1024]byte
		n, addr, err := listen.ReadFromUDP(data[:]) //接收数据
		if err != nil {
			fmt.Println("read udp failed,err:", err)
			continue
		}
		fmt.Printf("data:%v addr:%v count:%v\n", string(data[:n]), addr, n)
		_, err = listen.WriteToUDP(data[:n], addr)
		if err != nil {
			fmt.Println("write to udp failed,err:", err)
			continue
		}
	}
}

```

### UDPp客户端
如下
```golang
package main

import (
	"fmt"
	"net"
)

func main() {
	socket, err := net.DialUDP("udp", nil, &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("连接服务端失败，err:", err)
		return
	}
	defer socket.Close()
	sendData := []byte("Hello Server")
	_, err = socket.Write(sendData) //发送数据
	if err != nil {
		fmt.Println("发送数据失败，err:", err)
	}
	data := make([]byte, 4096)
	n, remoteAddr, err := socket.ReadFromUDP(data) //接收数据
	if err != nil {
		fmt.Println("接收数据失败，err:", err)
		return
	}
	fmt.Printf("recv:%v addr:%v count:%v\n", string(data[:n]), remoteAddr, n)
}

```

## Http编程
### http服务端
如下：
```golang
package main

import (
	"fmt"
	"net/http"
)

func main() {
	//单独写回调函数
	http.HandleFunc("/go", myHandler)

	//addr:监听地址
	//handler：回调函数
	http.ListenAndServe("127.0.0.1:8000", nil)
}

func myHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Println(r.RemoteAddr, "连接成功")
	fmt.Println("method:", r.Method)

	fmt.Println("url:", r.URL.Path)
	fmt.Println("header:", r.Header)
	fmt.Println("body:", r.Body)
	w.Write([]byte("你好: jarvenwang"))
}

```
### http客户端
```golang
package main

import (
	"fmt"
	"io"
	"net/http"
)

func main() {
	resp, _ := http.Get("http://127.0.0.1:8000/go")
	defer resp.Body.Close()

	fmt.Println(resp.Status)
	fmt.Println(resp.Header)

	buf := make([]byte, 1024)
	for {
		//接收服务端信息
		n, err := resp.Body.Read(buf)
		if err != nil && err != io.EOF {
			fmt.Println(err)
			return
		} else {
			fmt.Println("读取完毕")
			res := string(buf[:n])
			fmt.Println(res)
			break
		}
	}

}

```

## WebSocket编程
### WebSocket概念：
+ WebSocket是一种在单个TCP连接上进行全双工通信的协议
+ WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据
+ 在WebSocket API中，浏览器和服务器只要完成一次握手，两者之间就是可以创建持久性的连接，并进行双向数据传输
+ 安装第三方包：go get -u -v github.com/gorilla/websocket


------------------------------------------------
