## redis性能测试

### 语法

`bash
redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]
`

### 可选参数

|选项|描述|默认值|
|:---|:---|:---|
|-h <hostname> |     Server hostname (default 127.0.0.1)	|
|-p <port>     |     Server port (default 6379)	|
|-s <socket>   |     Server socket (overrides host and port)|
|-a <password> |     Password for Redis Auth|
|-c <clients>  |     Number of parallel connections (default 50)|
|-n <requests> |     Total number of requests (default 100000)|
|-d <size>     |     Data size of SET/GET value in bytes (default 2)|
|--dbnum <db>  |      SELECT the specified db number (default 0)|
|-k <boolean>  |     1=keep alive 0=reconnect (default 1)|
|-r <keyspacelen>|   Use random keys for SET/GET/INCR, random values for SADDUsing this option the benchmark willexpand the string __rand_int__inside an argument with a 12 digits number in the specified rangefrom 0 to keyspacelen-1. The substitution changes every time a commandis executed. Default tests use this to hit random keys ithe
specified range.|
|-P <numreq>   |     Pipeline <numreq> requests. Default 1 (no pipeline).|
|-e                 If server replies with errors, show them on stdout.               (no more than 1 error per second|
|-q            |     Quiet. Just show query/sec values|
|--csv         |     Output in CSV format|
|-l            |     Loop. Run the tests forever|
|-t <tests>    |     Only run the comma separated list of tests. The testnames are the same as the ones produced as |
|-I            |     Idle mode. Just open N idle connections and wait.|

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