## redis性能测试

### 语法

`bash
redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]
`

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
`bash
redis-benchmark
`

> Use 20 parallel clients, for a total of 100k requests, against 192.168.1.1:
`bash
redis-benchmark -h 192.168.1.1 -p 6379 -n 100000 -c 20
`

> Fill 127.0.0.1:6379 with about 1 million keys only using the SET test:
`bash
redis-benchmark -t set -n 1000000 -r 100000000
`

> Benchmark 127.0.0.1:6379 for a few commands producing CSV output:
`bash
redis-benchmark -t ping,set,get -n 100000 --csv
`

> Benchmark a specific command line:
`bash
redis-benchmark -r 10000 -n 10000 eval 'return redis.call("ping")' 0
`

> Fill a list with 10000 random elements:
`bash
redis-benchmark -r 10000 -n 10000 lpush mylist __rand_int__
`

 On user specified command lines __rand_int__ is replaced with a random integer
 with a range of values selected by the -r option.