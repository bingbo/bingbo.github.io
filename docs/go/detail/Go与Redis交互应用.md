

## 与redis的交互应用

### 安装redis客户端库

```bash
go get -u github.com/go-redis/redis
```

### 引入库建立连接并操作

```go
package main

import (
	"fmt"

	"github.com/go-redis/redis"
)

func main() {
	// 新建客户端连接
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "",
		DB:       0,
	})
	
	defer client.Close()
	// 连接服务器测试
	pong, err := client.Ping().Result()
	fmt.Println(pong, err)

	// 添加key-value
	err = client.Set("key1", "value1", 0).Err()
	if err != nil {
		panic(err)
	}

	// 获取存在的key
	val, err := client.Get("key1").Result()
	fmt.Println(val, err)

	// 获取不存在的key
	val2, err := client.Get("key2").Result()
	if err == redis.Nil {
		fmt.Println("key2 does not exist")
	} else if err != nil {
		panic(err)
	} else {
		fmt.Println("key2", val2)
	}
}
```


