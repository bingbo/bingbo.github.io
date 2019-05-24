参考[文档](https://tonybai.com/2015/11/17/tcp-programming-in-golang/)

## tcp连接数据超过最大量

### 问题

> 当server接收到的连接数据大于最大连接数据时会出现如下错误

```bash
accept err: accept tcp 0.0.0.0:8888: accept4: too many open files
```

> 当client打开的连接数据大于最大连接数时会出现如下错误

```bash
dial tcp 10.64.49.20:8888: socket: too many open files
```

### 解决方法

分别可通过ulimit -a查看或ulimit -n 100设置修改最大连接数

### 示例

> 客户端代码

```go
package main

import "fmt"
import "net"
import "time"

func main() {
    for i := 1; i < 1000; i++ {
        conn, err := net.Dial("tcp", "10.64.49.20:8888")
        if err != nil {
            fmt.Printf("%d: dial error:%s", i, err)
        }
        fmt.Println(i, ":connect to server ok")
        go handleConnection(conn)
    }
    time.Sleep(10000 * time.Second)
}

func handleConnection(c net.Conn) {
    defer c.Close()
    time.Sleep(10 * time.Second)
    fmt.Println("client handle done....")
}
```

> 服务端代码

```go
package main

import (
    "fmt"
    "net"
    "time"
)

func main() {

    l, err := net.Listen("tcp", ":8888")
    if err != nil {
        fmt.Println("listen err:", err)
        return
    }
    for {
        c, err := l.Accept()
        if err != nil {
            fmt.Println("accept err:", err)
            break
        }
        go handleConnection(c)
    }
}

func handleConnection(c net.Conn) {
    defer c.Close()
    time.Sleep(10 * time.Second)
    fmt.Println("server handle done....")
}
```

## 连接断开造成的问题

### 问题-连接在写入数据的时候，断开连接

> 结果

```bash
write tcp 172.24.198.180:52234->10.64.49.20:8888: write: broken pipe
```

> 示例

写入数据过程中，服务端断开连接

```go
// clent
conn, err := net.Dial("tcp", "10.64.49.20:8888")
if err != nil {
    fmt.Printf("dial error:%s", err)
    return
}
fmt.Println("connect to server ok")
defer conn.Close()
data := []byte("hello")

// 在写入数据的过程中服务端断开连接，write tcp 172.24.198.180:52421->10.64.49.20:8888: write: broken pipe
for i := 0; i < 100; i++ {
    time.Sleep(1 * time.Second)
    n, err := conn.Write(data)
    fmt.Println("write ", n, err)
    if err != nil {
        fmt.Printf("write %d bytes, error:%s\n", n, err)
        return
    }
}

fmt.Println("request done...")
```

```go
// server 启动稍后就关闭
package main

import (
	"fmt"
	"net"
	"time"
)

func main() {
	start := time.Now()
	l, err := net.Listen("tcp", ":8888")
	if err != nil {
		fmt.Println("listen err:", err)
		return
	}
	defer l.Close()
	for {
		c, err := l.Accept()
		if err != nil {
			fmt.Println("accept err:", err)
			return
		}
		fmt.Println("accept one")
		go handleConnection(c)

	}

	fmt.Println("take ", time.Since(start).Seconds())
}

func handleConnection(c net.Conn) {
	defer c.Close()
	// time.Sleep(3 * time.Second)
	// data := make([]byte, 1024)
	// c.Read(data)
	fmt.Println("server handle done.... ")
}
```

> 示例2

写入数据过程中，客户端断开连接

```go
// client
conn, err := net.Dial("tcp", "10.64.49.20:8888")
if err != nil {
    fmt.Printf("dial error:%s", err)
    return
}
fmt.Println("connect to server ok")
data := []byte("hello")

for i := 0; i < 10000; i++ {
    if i == 501 {
        time.Sleep(1 * time.Second)
        // 在写入数据的过程中断开连接
        conn.Close()
        break
    }
    time.Sleep(100 * time.Millisecond)
    n, err := conn.Write(data)
    //这里客户端连接已断开，继续写入报write tcp 172.24.198.180:52531->10.64.49.20:8888: write: broken pipe
    if err != nil {
        fmt.Printf("write %d bytes, error:%s\n", n, err)
        return
    }
    fmt.Println("write ", n, err)

}

fmt.Println("request done...")
```

## 连接被重置[connection reset by peer]

> 现象

```bash
write tcp 10.171.63.17:36484->10.176.86.163:8187: write: connection reset by peer
```

> 示例

