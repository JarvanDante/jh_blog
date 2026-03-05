---
author: "jarvan"
title: "gorilla/websocket"
date: 2022-12-05 00:00:01
description: "Golang官方认可的websocket库"
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
## 基础知识
### webSocket是什么
+ websocket是一种在单个tcp上进行 **全双工通信** 的协议，长连接，双向传输
+ 第三方包：go get -u -v github.com/gorilla/websocket
+ websocket协议实现起来相对简单。HTTP协议初始握手建立连接，websocket实质上使用原始TCP 读取/写入数据
+ http有良好的兼容性，ws和http的默认端口都是80，wss和https的默认端口都是443

### websocket握手协议
#### 客户端请求 Request Header
```text
    GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket   	// 指明使用WebSocket协议
    Connection: Upgrade		// 指明使用WebSocket协议
    Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==   // Bse64 encode的值，是浏览器随机生成的
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13  //指定Websocket协议版本
    Origin: http://example.com
```
说明：
服务端收到Sec-WebSocket-Key后拼接上一个固定的GUID，进行一次SHA-1摘要，再转成Base64编码，得到Sec-WebSocket-Accept返回给客户端。
客户端对本地的Sec-WebSocket-Key执行同样的操作跟服务端返回的结果进行对比，如果不一致会返回错误关闭连接。如此操作是为了把websocket header 跟http header区分开
#### 服务器响应 Response Header
```text
    HTTP/1.1 111 Switching Protocols
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: HSmrc1sMlYUkAGmm5OPpG2HaGWk=
    Sec-WebSocket-Protocol: chat
```

#### websocket发送的消息类型
5种：TextMessag、BinaryMessage、CloseMessag、PingMessage、PongMessage
+ TextMessag和BinaryMessage分别表示发送文本消息和二进制消息
+ CloseMessage关闭帧，接收方收到这个消息就关闭连接
+ PingMessage和PongMessage是保持心跳的帧，服务器发ping给浏览器，浏览器返回pong消息

#### 控制类消息
Websocket协议定义了三种控制消息：Close、Ping和Pong。
通过调用Conn的WriteControl、WriteMessage或NextWriter方法向对端发送控制消息

Conn收到了Close消息之后，调用由SetCloseHandler方法设置的handler函数，然后从NextReader 、ReadMessage或消息的Read 方法返回一个*CloseError。缺省的close handler会发送一个Close消息到对端。

Conn收到了Ping消息之后，调用由SetPingHandler 方法设置的handler函数。缺省的ping handler会发送一个Pong消息到对象。

Conn收到了Pong消息之后，调用由SetPongHandler 设置的handler函数。缺省的pong handler什么也不做。

控制消息的handler函数是从NextReader、ReadMessage和消息的Read方法中调用的。缺省的close handler和ping handler向对端写数据时可能会短暂阻塞这些方法。

应用程序必须读取Conn，使得对端发送的close、ping、和pong消息能够得到处理。即使应用程序不关心对端发送的消息，也应该启动一个goroutine来读取对端的消息并丢弃。例如：


## gorilla/websocket
websocket由http升级而来，首先发送附带Upgrade请求头的Http请求，所以我们需要在处理Http请求时拦截请求并判断其是否为websocket升级请求，如果是则调用gorilla/websocket库相应函数处理升级请求

### Upgrader
Upgrader发送附带Upgrade请求头的Http请求,把 http 请求升级为长连接的 WebSocket,结构如下：
```go
type Upgrader struct {
    // 升级 websocket 握手完成的超时时间
    HandshakeTimeout time.Duration

    // io 操作的缓存大小，如果不指定就会自动分配。
    ReadBufferSize, WriteBufferSize int

    // 写数据操作的缓存池，如果没有设置值，write buffers 将会分配到链接生命周期里。
    WriteBufferPool BufferPool

    //按顺序指定服务支持的协议，如值存在，则服务会从第一个开始匹配客户端的协议。
    Subprotocols []string

    // http 的错误响应函数，如果没有设置 Error 则，会生成 http.Error 的错误响应。
    Error func(w http.ResponseWriter, r *http.Request, status int, reason error)

    // 如果请求Origin标头可以接受，CheckOrigin将返回true。 如果CheckOrigin为nil，则使用安全默认值：如果Origin请求头存在且原始主机不等于请求主机头，则返回false。
    // 请求检查函数，用于统一的链接检查，以防止跨站点请求伪造。如果不检查，就设置一个返回值为true的函数
    CheckOrigin func(r *http.Request) bool

    // EnableCompression 指定服务器是否应尝试协商每个邮件压缩（RFC 7692）。 将此值设置为true并不能保证将支持压缩。 目前仅支持“无上下文接管”模式
    EnableCompression bool
}
```
### 创建Upgrader实例
该实例用于升级请求
```go
var upgrader = websocket.Upgrader{
    ReadBufferSize:  1124, //指定读缓存大小
    WriteBufferSize: 1124, //指定写缓存大小
    CheckOrigin:     checkOrigin,
}
// 检测请求来源
func checkOrigin(r *http.Request) bool {
    if r.Method != "GET" {
        fmt.Println("method is not GET")
        return false
    }
    if r.URL.Path != "/ws" {
        fmt.Println("path error")
        return false
    }
    return true
}
```
### 升级协议
func (*Upgrader) Upgrade 函数将 http 升级到 WebSocket 协议。
```go
// responseHeader包含在对客户端升级请求的响应中。 
// 使用responseHeader指定cookie（Set-Cookie）和应用程序协商的子协议（Sec-WebSocket-Protocol）。
// 如果升级失败，则升级将使用HTTP错误响应回复客户端
// 返回一个 Conn 指针，使用 Conn 读写数据与客户端通信。
func (u *Upgrader) Upgrade(w http.ResponseWriter, r *http.Request, responseHeader http.Header) (*Conn, error)
```

升级为websocket连接并获得一个conn实例，之后的发送接收操作皆有conn，其类型为websocket.Conn。
```go
//Http入口
func (e *Engine) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    //判断请求是否为websocket升级请求。
    if websocket.IsWebSocketUpgrade(r) {
        // 收到 http 请求后升级协议
        conn, err := upgrader.Upgrade(w, r, w.Header())
        // 向客户端发送消息使用 WriteMessage(messageType int, data []byte),参数1为消息类型，参数2消息内容
        conn.WriteMessage(websocket.TextMessage, []byte("升级成功"))
        // 接受客户端消息使用 ReadMessage(),该操作阻塞线程所以建议运行在其他协程上。
        //返回值(接收消息类型、接收消息内容、发生的错误)当然正常执行时错误为 nil。一旦连接关闭返回值类型为-1可用来终止读操作。
        go func() {
            for {
                t, c, _ := conn.ReadMessage()
                fmt.Println(t, string(c))
                if t == -1 {
                    return
                }
            }
        }()
    } else {
        //处理普通请求
        c := newContext(w, r)
        e.router.handle(c)
    }
}
```
### 设置关闭连接监听
函数为SetCloseHandler(h func(code int, text string) error)函数接收一个函数为参数，参数为nil时有一个默认实现，其源码为：

```go
func (c *Conn) SetCloseHandler(h func(code int, text string) error) {
    if h == nil {
        h = func(code int, text string) error {
            message := FormatCloseMessage(code, "")
            c.WriteControl(CloseMessage, message, time.Now().Add(writeWait))
            return nil
        }
    }
    c.handleClose = h
}
```

可以看到作为参数的函数的参数为int和string类型正好和前端的close(long string)对应即前端调用close(long string)关闭连接后两个参数会被发送给后端并最终被func(code int, text string) error所使用。

```go
// 设置关闭连接监听
conn.SetCloseHandler(func(code int, text string) error {
    fmt.Println(code, text) // 断开连接时将打印code和text
    return nil
})
```

### 总览
```go
type WsServer struct {
    ......
    // 定义一个 upgrade 类型用于升级 http 为 websocket
    upgrade  *websocket.Upgrader
}

func NewWsServer() *WsServer {
    ws.upgrade = &websocket.Upgrader{
        ReadBufferSize:  4196,//指定读缓存区大小
        WriteBufferSize: 1124,// 指定写缓存区大小
        // 检测请求来源
        CheckOrigin: func(r *http.Request) bool {
            if r.Method != "GET" {
                fmt.Println("method is not GET")
                return false
            }
            if r.URL.Path != "/ws" {
                fmt.Println("path error")
                return false
            }
            return true
        },upgrade
    }
    return ws
}

func (self *WsServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    ......
    // 收到 http 请求后 升级 协议
    conn, err := self.upgrade.Upgrade(w, r, nil)
    if err != nil {
        fmt.Println("websocket error:", err)
        return
    }
    fmt.Println("client connect :", conn.RemoteAddr())
    go self.connHandle(conn)

}
```
## Demo
### 复杂版本
#### 服务端：
```go
package main

import (
	"fmt"
	"github.com/gorilla/websocket"
	"log"
	"net/http"
)

var upgrader = websocket.Upgrader{
	ReadBufferSize:  4196,
	WriteBufferSize: 1124,
	CheckOrigin: func(r *http.Request) bool {

		//if r.Method != "GET" {
		//	fmt.Println("method is not GET")
		//	return false
		//}
		//if r.URL.Path != "/ws" {
		//	fmt.Println("path error")
		//	return false
		//}
		return true
	},
}

// ServerHTTP 用于升级协议
func ServerHTTP(w http.ResponseWriter, r *http.Request) {
	// 收到http请求之后升级协议
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("Error during connection upgrade:", err)
		return
	}
	defer conn.Close()

	for {
		// 服务端读取客户端请求
		messageType, message, err := conn.ReadMessage()
		if err != nil {
			log.Println("Error during message reading:", err)
			break
		}
		log.Printf("Received:%s", message)

		// 开启关闭连接监听
		conn.SetCloseHandler(func(code int, text string) error {
			fmt.Println(code, text) // 断开连接时将打印code和text
			return nil
		})

		//服务端给客户端返回请求
		err = conn.WriteMessage(messageType, message)
		if err != nil {
			log.Println("Error during message writing:", err)
			return
		}

	}
}

func home(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Index Page")
}

func main() {
	http.HandleFunc("/socket", ServerHTTP)
	http.HandleFunc("/", home)
	log.Fatal(http.ListenAndServe("localhost:8181", nil))
}
```
### 客户端：
```go
// client.go
package main

import (
	"github.com/gorilla/websocket"
	"log"
	"os"
	"os/signal"
	"time"
)

var done chan interface{}
var interrupt chan os.Signal

func receiveHandler(connection *websocket.Conn) {
	defer close(done)
	for {
		_, msg, err := connection.ReadMessage()
		if err != nil {
			log.Println("Error in receive:", err)
			return
		}
		log.Printf("Received: %s\n", msg)
	}
}

func main() {
	done = make(chan interface{})    // Channel to indicate that the receiverHandler is done
	interrupt = make(chan os.Signal) // Channel to listen for interrupt signal to terminate gracefully

	signal.Notify(interrupt, os.Interrupt) // Notify the interrupt channel for SIGINT

	socketUrl := "ws://localhost:8181" + "/socket"
	conn, _, err := websocket.DefaultDialer.Dial(socketUrl, nil)
	if err != nil {
		log.Fatal("Error connecting to Websocket Server:", err)
	}
	defer conn.Close()
	go receiveHandler(conn)

	// 无限循环使用select来通过通道监听事件
	for {
		select {
		case <-time.After(time.Duration(1) * time.Millisecond * 1111):
			//conn.WriteMessage()每秒钟写一条消息
			err := conn.WriteMessage(websocket.TextMessage, []byte("Hello from GolangDocs!"))
			if err != nil {
				log.Println("Error during writing to websocket:", err)
				return
			}
		//如果激活了中断信号，则所有未决的连接都将关闭
		case <-interrupt:
			// We received a SIGINT (Ctrl + C). Terminate gracefully...
			log.Println("Received SIGINT interrupt signal. Closing all pending connections")

			// Close our websocket connection
			err := conn.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseNormalClosure, ""))
			if err != nil {
				log.Println("Error during closing websocket:", err)
				return
			}

			select {
			// 如果receiveHandler通道退出，则通道'done'将关闭
			case <-done:
				log.Println("Receiver Channel Closed! Exiting....")
			//如果'done'通道未关闭，则在1秒钟后会有超时，因此程序将在1秒钟超时后退出
			case <-time.After(time.Duration(1) * time.Second):
				log.Println("Timeout in closing receiving channel. Exiting....")
			}
			return
		}
	}
}
```

### 简单版
#### 服务端：
```go
package main

import (
	"github.com/gorilla/websocket"
	"log"
	"net/http"
)

var upgrade = websocket.Upgrader{
	ReadBufferSize:  1124,
	WriteBufferSize: 1124,
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
}

func HelloHTTP(w http.ResponseWriter, r *http.Request) {
	//1.升级协议，并返回升级后的长连接
	conn, err := upgrade.Upgrade(w, r, nil)

	if err != nil {
		log.Println("Error during connection upgrade:", err)
		return
	}
	defer conn.Close()

	for {
		// 2.读取客户端的请求信息
		messageType, message, err := conn.ReadMessage()
		if err != nil {
			log.Println("Error during message writing:", err)
			return
		}
		log.Printf("Recive message:%s", message)
		//	3.返回给客户端信息
		err = conn.WriteMessage(messageType, message)
		if err != nil {
			log.Println("Error during message writing:", err)
			return
		}
	}

}

func main() {

	http.HandleFunc("/socket", HelloHTTP)
	http.ListenAndServe(":8181", nil)
}
```

#### 客户端：
```go
package main

import (
   "github.com/gorilla/websocket"
   "log"
   "time"
)

func ReceiveHandler(con *websocket.Conn) {
   for {
      _, message, err := con.ReadMessage()
      if err != nil {
         log.Println("Error during Receive:", err)
         return
      }
      log.Printf("Receive:%s\n", message)
   }
}

func main() {
   socketUrl := "ws://localhost:8181" + "/socket"
   // 使用 net.Dialer Dialer.Dial 函数建立 TCP 连接,建立成功后，取得了 net.Conn 对象，
   conn, _, err := websocket.DefaultDialer.Dial(socketUrl, nil)
   if err != nil {
      log.Fatal("Error connecting to websocket Server:", err)
   }
   defer conn.Close()

   ticker := time.Tick(time.Second)
   for range ticker {
      err = conn.WriteMessage(websocket.TextMessage, []byte("Hello World!"))
      if err != nil {
         log.Println("Error during writing to websocket:", err)
         return
      }
      // 接受客户端消息使用ReadMessage()该操作会阻塞线程所以建议运行在其他协程上
      go ReceiveHandler(conn)
   }
}
```


## Gin框架结合gorilla
### 安装Gin和gorilla
```shell
go get -u github.com/gin-gonic/gin #如果下载慢看看是否用代理 或 go mod init 模块名
go get -u github.com/gorilla/websocket
```
### demo
```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/gorilla/websocket"
	"log"
	"net/http"
)

var upgrader = websocket.Upgrader{
	CheckOrigin: func(r *http.Request) bool {
		// 在这里进行权限验证，返回true表示验证通过，允许连接
		// 你可以根据需要实现自己的验证逻辑
		return true
	},
}

func main() {
	r := gin.Default()

	// 提供一个HTTP端点用于升级连接为WebSocket
	r.GET("/ws", func(c *gin.Context) {
		serveWebSocket(c.Writer, c.Request)
	})

	r.Run(":8085")
}

func serveWebSocket(w http.ResponseWriter, r *http.Request) {
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println(err)
		return
	}
	defer conn.Close()

	for {
		// 读取客户端发送的消息
		messageType, p, err := conn.ReadMessage()
		if err != nil {
			log.Println(err)
			return
		}
		log.Println("来自客户端的信息：")
		log.Println(string(p))
		// 将消息原样发送回客户端
		err = conn.WriteMessage(messageType, []byte("我收到你的信息了"))
		if err != nil {
			log.Println(err)
			return
		}
	}
}
```
### 调试
```shell
go run main.go
```
使用Apifox -> websocket

### nginx ssl证书进行升级
```text
server {
    listen 443 ssl;
    server_name your_domain.com
    ssl_certificate /path/to/your_cert.crt;
    ssl_certificate_key /path/to/your_private_key.key;
    
    location / {
        proxy_pass http://127.0.0.1:8080; # 这里是你的Gin应用的地址和端口
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```