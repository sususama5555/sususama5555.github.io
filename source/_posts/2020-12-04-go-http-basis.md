---
title: Go语言HTTP编程
date: 2020-05-15 00:32:26
tags: 
- Golang
- HTTP
- 网络编程
- 代码实践
categories: 
- Golang
- 网络编程
---

# 概述

## Web工作方式

我们平时浏览网页的时候，会打开浏览器，输入网址后按下回车键，然后会显示你想要浏览的内容。在这个看似简单的用户行为背后，到底隐藏了些什么呢？？

对于普通的上网过程，过程为：浏览器本身是一个客户端，当输入URL的时候，首先浏览器会去请求
DNS服务器。

通过DNS获取相应的域名对应的IP，然后通过IP地址找到IP对应的服务器后，要求建立TCP连接。

等浏览器发送完HTTPRequest(请求)包后，服务器接收到请求包开始处理请求包。服务器调用自身服
务，返回HTTPResponse(响应)包。

客户端收到来看服务器的响应后开始渲染这个Response包里的主体(body)，等收到全部的内容随后断开与该服务器之间的TCP连接。

一个Web服务器也被称为HTTP服务器，它通过HTTP协议与客户端通信。这个客户端通常指的是Web浏览器(包括手机上的各种客户端))

Web服务器的工作原理可以简单的归纳为：

- 客户端通过TCP/IP协议建立到服务器的TCP连接
- 客户端向服务器发送HTTP协议请求包，请求服务器里的资源文档
- 服务器向客户端发送HTTP协议应答包
- 客户端与服务器断开连接，客户端解释HTML文档，在客户端屏幕上渲染图形结果

<!--more-->

## HTTP协议

HTTP协议的全称：超文本传输协议，HyperText Transfer Protocol。

HTTP协议是互联网上应用最为广泛的一种网络协议，详细规定了浏览器和万维网服务器之间互相通信的规则，通过因特网传递万维网文档的数据传送协议。

HTTP协议通常承载于TCP协议之上，有时也承载于TLS或SSL协议层之上，这个时候就成了常说的
HTTPS。

## 地址(URL)

URL全称为：Unique Resource Location，用来表示网络资源，可以理解为网络文件路径。

URL的格式如下：

```http
http://host[":"port][/abs_path]
http://192.168.1.1/html/index
```

URL的长度有限制，不同的服务器的限制值太太相同，但是不能无限长。

## 请求报文

HTTP请求报文由请求行，请求头部，请求包体等组成。
```go
package main

import (
	"fmt"
	"net"
)

func main() {
	//监听
	listener, err := net.Listen("tcp", ":8000")
	if err != nil {
		fmt.Println("Listen err = ", err)
		return
	}
	defer listener.Close()
	//阻塞等待用户的连接
	conn, err1 := listener.Accept()
	if err1 != nil {
		fmt.Println("Accept err1 = ", err1)
		return
	}
	defer conn.Close()
	//接收客户端的数据
	buf := make([]byte, 1024*4)
	n, err2 := conn.Read(buf)
	if n == 0 {
		fmt.Println("Read err = ", err2)
		return
	}
	fmt.Printf("#%v#", string(buf[:n]))
}
```
运行程序，用浏览器请求URL： http://127.0.0.1:8000/ ,程序后台打印结果：

``` http
#GET / HTTP/1.1
Host: 127.0.0.1:8000
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML,like Gecko) Chrome/65.0.3325.146 Safari/537.36
Accept:
text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
```


## 响应数据

```go
package main

import (
	"fmt"
	"net/http"
)

//服务端编写的业务逻辑处理程序
func myHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "hello world")
}
func main() {
	http.HandleFunc("/go", myHandler)
	//在指定的地址进行监听，开启一个HTTP
	http.ListenAndServe("127.0.0.1:8000", nil)
}

```

运行程序，用浏览器请求URL： http://127.0.0.1:8000/go ,程序会响应"hello world"在浏览器页面
上。

## 响应报文格式

服务端代码：
```go
package main

import (
	"fmt"
	"net/http"
)

//服务端编写的业务逻辑处理程序
func myHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "hello world")
}
func main() {
	http.HandleFunc("/go", myHandler)
	//在指定的地址进行监听，开启一个HTTP
	http.ListenAndServe("127.0.0.1:8000", nil)
}

```



客户端代码：

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	//主动连接服务器
	conn, err := net.Dial("tcp", ":8000")
	if err != nil {
		fmt.Println("dial err = ", err)
		return
	}
	defer conn.Close()
	requestBuf := "GET /go HTTP/1.1\r\nAccept: image/gif, image/jpeg,
image/pjpeg, application/x-ms-application, application/xaml+xml, application/xms-xbap, */*\r\nAccept-Language: zh-Hans-CN,zh-Hans;q=0.8,enUS;q=0.5,en;q=0.3\r\nUser-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT10.0; WOW64; Trident/7.0; .NET4.0C; .NET4.0E; .NET CLR 2.0.50727; .NET CLR3.0.30729; .NET CLR 3.5.30729)\r\nAccept-Encoding: gzip, deflate\r\nHost:127.0.0.1:8000\r\nConnection: Keep-Alive\r\n\r\n"
	//先发请求包，服务器才会回响应包
	conn.Write([]byte(requestBuf))
	//接收服务器回复的响应包
	buf := make([]byte, 1024*4)
	n, err1 := conn.Read(buf)
	if n == 0 {
		fmt.Println("Read err = ", err1)
		return
	}
	//打印响应报文
	fmt.Printf("#%v#", string(buf[:n]))
}

```

先运行服务端程序，再运行客户端程序，客户端程序得到结果如下：

```http
#HTTP/1.1 200 OK
Date: Thu, 09 Apr 2020 13:07:35 GMT
Content-Length: 12
Content-Type: text/plain; charset=utf-8
hello world
```

## HTTP状态码

状态码由三位数据组成，第一位数据表示响应的类型，常用的状态码有五大类：

| 状态码 | 含义                                                 |
| :----- | ---------------------------------------------------- |
| 1xx    | 表示服务器已接收到客户端的请求，客户端可继续发送请求 |
| 2xx    | 表示服务器已成功接收到请求，并进行处理               |
| 3xx    | 表示服务器要求客户端重定向                           |
| 4xx    | 表示客户端的请求有非法内容                           |
| 5xx    | 表示服务器未能正常处理客户端的请求而出现意外错误     |

常见的状态码举例：

```
200 OK：客户端请求成功
400 Bad Request：请求报文有语法错误
401 Unauthorized：未授权
403 Forbidden：服务器拒绝服务
404 Not Found：请求资源不存在
500 Internal Server Error：服务器内部错误
502 bad Gateway: 网关错误
503 Server Unavailable：服务器临时不能处理客户端请求
```

# HTTP编程

Go请求标准库内建提供了net/http包，涵盖了HTTP客户端和服务器的具体实现。
使用net/http包可以很方便的编写HTTP客户端或服务端程序。

## HTTP服务端
```go
package main

import (
	"fmt"
	"net/http"
)

//w, 给客户端回复数据
//r, 读取客户端发送的数据
func HandConn(w http.ResponseWriter, r *http.Request) {
	fmt.Println("r.Method = ", r.Method)
	fmt.Println("r.URL = ", r.URL)
	fmt.Println("r.Header = ", r.Header)
	fmt.Println("r.Body = ", r.Body)
	w.Write([]byte("hello go")) //给客户端回复数据
}
func main() {
	//注册处理函数，用户连接，自动调用指定的处理函数
	http.HandleFunc("/go", HandConn)
	//监听绑定
	http.ListenAndServe(":8000", nil)
}

```

运行程序后，用浏览器打开URL： http://127.0.0.1:8000/go ,页面显示"hello go"
服务端运行结果：

```http
r.Method = GET
r.URL = /go
r.Header = map[Connection:[keep-alive] Cache-Control:[max-age=0] UpgradeInsecure-Requests:[1] User-Agent:[Mozilla/5.0 (Windows NT 10.0; Win64; x64)
AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36]
Accept:
[text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8] Accept-Encoding:[gzip, deflate, br] Accept-Language:[zhCN,zh;q=0.9,en;q=0.8]]
r.Body = {}
```



## HTTP客户端

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	resp, err := http.Get("http://127.0.0.1:8000/go")
	if err != nil {
		fmt.Println("http.Get err = ", err)
		return
	}
	defer resp.Body.Close()
	fmt.Println("Status = ", resp.Status)
	fmt.Println("StatusCode = ", resp.StatusCode)
	fmt.Println("Header = ", resp.Header)
	//fmt.Println("Body = ", resp.Body)
	buf := make([]byte, 4*1024)
	var tmp string
	for {
		n, err := resp.Body.Read(buf)
		if n == 0 {
			fmt.Println("read err = ", err)
			break
		}
		tmp += string(buf[:n])
	}
	fmt.Println("tmp = ", tmp)
}
```

运行客户端程序，会自动请求"http://127.0.0.1:8000/go"这个地址，此时客户端后台打印：

```http
Status = 200 OK
StatusCode = 200
Header = map[Date:[Thu, 09 Apr 2020 13:25:54 GMT] Content-Length:[8] ContentType:[text/plain; charset=utf-8]]
read err = EOF
tmp = hello go
```

此时服务端后台打印结果：
```go
r.Method = GET
r.URL = /go
r.Header = map[User-Agent:[Go-http-client/1.1] Accept-Encoding:[gzip]]
r.Body = {}
```