## redis性能测试

### 语法

```bash
redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]
```

> 如默认配置：

```bash
$redis-benchmark
====== PING_INLINE ======
  100000 requests completed in 1.85 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

97.81% <= 1 milliseconds
99.80% <= 2 milliseconds
99.96% <= 3 milliseconds
100.00% <= 3 milliseconds
54200.54 requests per second

====== PING_BULK ======
  100000 requests completed in 2.00 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

98.05% <= 1 milliseconds
99.94% <= 2 milliseconds
100.00% <= 3 milliseconds
50050.05 requests per second
```

### 可选参数

|选项|描述|默认值|
|:---|:---|:---|
|-h <hostname> |     指定服务器主机名|127.0.0.1	|
|-p <port>     |     指定服务器端口|6379	|
|-s <socket>   |     服务器socket，覆盖host和port|-|
|-a <password> |     redis验证密码|-|
|-c <clients>  |     指定并发连接数|50|
|-n <requests> |     指定请求数|100000|
|-d <size>     |     以字节的形式指定 SET/GET 值的数据大小|2|
|--dbnum <db>  |      SELECT the specified db number (default 0)|
|-k <boolean>  |     是否保持连接，1=keep alive 0=reconnect| 1|
|-r <keyspacelen>|   SET/GET/INCR 使用随机 key, SADD 使用随机值|-|
|-P <numreq>   |     通过管道传输 <numreq> 请求|1|
|-e            |     如果服务器返回错误，则在控制台输出显示|-|
|-q            |     强制退出 redis。仅显示 query/sec 值|-|
|--csv         |     按CSV格式输出|-|
|-l            |     循环，永久执行测试|-|
|-t <tests>    |     仅运行以逗号分隔的测试命令列表|-|
|-I            |     Idle 模式。仅打开 N 个 idle 连接并等待|-|

### 示例

> 按默认配置在127.0.0.1:6379上执行

```bash
redis-benchmark
```

> 20个并发，100000个请求，在192.168.1.1:6379上执行

```bash
redis-benchmark -h 192.168.1.1 -p 6379 -n 100000 -c 20
```

> 在127.0.0.1:6379用随机1000000个key测试set命令

```bash
redis-benchmark -t set -n 1000000 -r 100000000
```

> 在127.0.0.1:6379测试指定一些命令并以csv格式输出

```bash
redis-benchmark -t ping,set,get -n 100000 --csv
```

> 测试指定命令行

```bash
redis-benchmark -r 10000 -n 10000 eval 'return redis.call("ping")' 0
```

> 使用10000个随机数填充一个list，可以用`__rand_int__`代替-r参数产生随机数

```bash
redis-benchmark -r 10000 -n 10000 lpush mylist __rand_int__
```
