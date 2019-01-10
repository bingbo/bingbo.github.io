通过redis-cli命令来管理服务器及排查故障，参考[官网](https://redis.io/topics/rediscli)

> 命令行用法

```bash
$ redis-cli incr mycounter
(integer) 7
```

> 指定服务器执行命令

```bash
$ redis-cli -h redis15.localnet.org -p 6390 ping
PONG
```

> 从其他地方获取输入

```bash
$ redis-cli -x set foo < /etc/services
OK
```

> 连续运行相同命令

```bash
$ redis-cli -r 5 incr foo
(integer) 1
(integer) 2
(integer) 3
(integer) 4
(integer) 5
```

> 连续统计实时监控redis实例，用--stat选项

```bash
$ redis-cli --stat
------- data ------ --------------------- load -------------------- - child -
keys       mem      clients blocked requests            connections
506        1015.00K 1       0       24 (+0)             7
506        1015.00K 1       0       25 (+1)             7
506        3.40M    51      0       60461 (+60436)      57
506        3.40M    51      0       146425 (+85964)     107
507        3.40M    51      0       233844 (+87419)     157
507        3.40M    51      0       321715 (+87871)     207
508        3.40M    51      0       408642 (+86927)     257
508        3.40M    51      0       497038 (+88396)     257
```

_-i <interval>选项用作修饰符，以便更改发出新行的频率。默认值为一秒_

> 扫描其中的大键及数据类型信息，用--bigkeys选项

```bash
$ redis-cli --bigkeys

# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).

[00.00%] Biggest string found so far 'key-419' with 3 bytes
[05.14%] Biggest list   found so far 'mylist' with 100004 items
[35.77%] Biggest string found so far 'counter:__rand_int__' with 6 bytes
[73.91%] Biggest hash   found so far 'myobject' with 3 fields
```

> 扫描相关的key，使用--scan选项

```bash
$ redis-cli --scan | head -10
key-419
key-71
key-236

#指定匹配项
$ redis-cli --scan --pattern '*-11*'
key-114
key-117
key-118
key-113

#统计数量
$ redis-cli --scan --pattern 'user:*' | wc -l
3829433
```

> 使用MONITOR模式，自动进入监控模式。它将打印Redis实例接收的所有命令

```bash
$ redis-cli monitor
OK
1460100081.165665 [0 127.0.0.1:51706] "set" "foo" "bar"
1460100083.053365 [0 127.0.0.1:51707] "get" "foo"
```

> 监控Redis实例的延迟

```bash
$ redis-cli --latency
min: 0, max: 1, avg: 0.19 (427 samples)
```

_基本延迟检查工具是--latency可选的。使用此选项，CLI将运行一个循环，将PING命令发送到Redis实例，并测量获得答复的时间。这种情况每秒发生100次，并且统计数据在控制台中实时更新_

> RDB文件的远程备份

```bash
$ redis-cli --rdb /tmp/dump.rdb
SYNC sent to master, writing 13256 bytes to '/tmp/dump.rdb'
Transfer finished with success.
```

_在Redis复制的第一次同步期间，主服务器和从服务器以RDB文件的形式交换整个数据集。此功能被利用redis-cli以提供远程备份工具，允许将RDB文件从任何Redis实例传输到运行的本地计算机_

