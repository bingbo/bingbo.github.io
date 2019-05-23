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
