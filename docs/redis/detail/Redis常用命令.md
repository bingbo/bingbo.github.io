## redis中常用命令

其他所有命令见[官网](https://redis.io/commands)

### redis中常用的管理服务的命令

#### info

返回关于服务器的各种信息及统计数值

> 语法

```bash
INFO [section]
```

> 示例

```bash
127.0.0.1:6379> info
# Server
redis_version:3.2.11
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:bbb8887306f6caae
redis_mode:standalone
os:Darwin 18.2.0 x86_64
arch_bits:64
multiplexing_api:kqueue
gcc_version:4.2.1
process_id:45048
run_id:678f90f316738b90f32bf0af46882d99abc89be1
tcp_port:6379
uptime_in_seconds:26445
uptime_in_days:0
hz:10
lru_clock:3607929
executable:/Users/zhangbingbing/redis-server
config_file:

# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:1186720
used_memory_human:1.13M
used_memory_rss:1236992
used_memory_rss_human:1.18M
used_memory_peak:5354558896
used_memory_peak_human:4.99G
total_system_memory:8589934592
total_system_memory_human:8.00G
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:1.04
mem_allocator:libc

# Persistence
loading:0
rdb_changes_since_last_save:1
rdb_bgsave_in_progress:0
rdb_last_save_time:1547108387
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok

# Stats
total_connections_received:3165
total_commands_processed:2518592
instantaneous_ops_per_sec:0
total_net_input_bytes:986488961
total_net_output_bytes:1335294340
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
evicted_keys:0
keyspace_hits:605073
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:996
migrate_cached_sockets:0

# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:1174.07
used_cpu_user:631.43
used_cpu_sys_children:1473.40
used_cpu_user_children:1008.22

# Cluster
cluster_enabled:0

# Keyspace
db0:keys=2,expires=0,avg_ttl=0

###只返回内存占用信息
127.0.0.1:6379> INFO Memory
# Memory
used_memory:1185904
used_memory_human:1.13M
used_memory_rss:1302528
used_memory_rss_human:1.24M
used_memory_peak:5354558896
used_memory_peak_human:4.99G
total_system_memory:8589934592
total_system_memory_human:8.00G
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:1.10
mem_allocator:libc
```

> 参数说明

* server : 一般 Redis 服务器信息，包含以下域：

    * redis_version : Redis 服务器版本
    * redis_git_sha1 : Git SHA1
    * redis_git_dirty : Git dirty flag
    * os : Redis 服务器的宿主操作系统
    * arch_bits : 架构（32 或 64 位）
    * multiplexing_api : Redis 所使用的事件处理机制
    * gcc_version : 编译 Redis 时所使用的 GCC 版本
    * process_id : 服务器进程的 PID
    * run_id : Redis 服务器的随机标识符（用于 Sentinel 和集群）
    * tcp_port : TCP/IP 监听端口
    * uptime_in_seconds : 自 Redis 服务器启动以来，经过的秒数
    * uptime_in_days : 自 Redis 服务器启动以来，经过的天数
    * lru_clock : 以分钟为单位进行自增的时钟，用于 LRU 管理
    
* clients : 已连接客户端信息，包含以下域：

    * connected_clients : 已连接客户端的数量（不包括通过从属服务器连接的客户端）
    * client_longest_output_list : 当前连接的客户端当中，最长的输出列表
    * client_longest_input_buf : 当前连接的客户端当中，最大输入缓存
    * blocked_clients : 正在等待阻塞命令（BLPOP、BRPOP、BRPOPLPUSH）的客户端的数量
    
* memory : 内存信息，包含以下域：

    * used_memory : 由 Redis 分配器分配的内存总量，以字节（byte）为单位
    * used_memory_human : 以人类可读的格式返回 Redis 分配的内存总量
    * used_memory_rss : 从操作系统的角度，返回 Redis 已分配的内存总量（俗称常驻集大小）。这个值和 top 、 ps 等命令的输出一致。
    * used_memory_peak : Redis 的内存消耗峰值（以字节为单位）
    * used_memory_peak_human : 以人类可读的格式返回 Redis 的内存消耗峰值
    * used_memory_lua : Lua 引擎所使用的内存大小（以字节为单位）
    * mem_fragmentation_ratio : used_memory_rss 和 used_memory 之间的比率
    * mem_allocator : 在编译时指定的， Redis 所使用的内存分配器。可以是 libc 、 jemalloc 或者 tcmalloc 。
    
    在理想情况下， used_memory_rss 的值应该只比 used_memory 稍微高一点儿。
    当 rss > used ，且两者的值相差较大时，表示存在（内部或外部的）内存碎片。
    内存碎片的比率可以通过 mem_fragmentation_ratio 的值看出。
    当 used > rss 时，表示 Redis 的部分内存被操作系统换出到交换空间了，在这种情况下，操作可能会产生明显的延迟。
    当 Redis 释放内存时，分配器可能会，也可能不会，将内存返还给操作系统。
    如果 Redis 释放了内存，却没有将内存返还给操作系统，那么 used_memory 的值可能和操作系统显示的 Redis 内存占用并不一致。
    查看 used_memory_peak 的值可以验证这种情况是否发生。

* persistence : RDB 和 AOF 的相关信息
* stats : 一般统计信息
* replication : 主/从复制信息
* cpu : CPU 计算量统计信息
* commandstats : Redis 命令统计信息
* cluster : Redis 集群信息
* keyspace : 数据库相关的统计信息

#### DBSIZE

返回数据库中key的数量

```bash
127.0.0.1:6379> DBSIZE
(integer) 2
```

#### FLUSHALL

清空整个 Redis 服务器的数据(删除所有数据库的所有 key )

```bash
127.0.0.1:6379> FLUSHALL
OK
```

#### FLUSHDB

清空当前数据库中的所有 key

```bash
127.0.0.1:6379> FLUSHDB
OK
```

#### CONFIG GET parameter 
    
获取指定配置参数的值

#### CONFIG REWRITE 

对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写

#### CONFIG SET parameter value 

修改 redis 配置参数，无需重启

#### CONFIG RESETSTAT 

重置 INFO 命令中的某些统计数据

#### LASTSAVE 

返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示

#### MONITOR 

实时打印出 Redis 服务器接收到的命令，调试用