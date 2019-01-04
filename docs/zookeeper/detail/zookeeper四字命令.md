### conf

> conf命令用于输出zookeeper服务器运行时使用的基本配置信息，包括clientPort,dataDir和tickTIme等

### cons

> 用于输出当前这台服务器上所有客户端连接的详细信息，包括每个客户端的IP，会话ID和最后一次与服务器交互的操作类型等

### crst

> 用于重置所有的客户端连接统计信息

### dump

> 用于输出当前集群的所有会话信息，包括这些会话的ID及创建的临时节点等信息

### envi

> 用于输出zookeeper所在服务器运行时的环境信息，包括os.version,java.version和user.home等

```
$telnet 127.0.0.1 2181
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
envi
Environment:
zookeeper.version=3.4.9-1757313, built on 08/23/2016 06:50 GMT
host.name=30.28.177.217
java.version=1.8.0_101
java.vendor=Oracle Corporation
java.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre
java.class.path=/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../build/classes:/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../build/lib/*.jar:/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../lib/slf4j-log4j12-1.6.1.jar:/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../lib/slf4j-api-1.6.1.jar:/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../lib/netty-3.10.5.Final.jar:/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../lib/log4j-1.2.16.jar:/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../lib/jline-0.9.94.jar:/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../zookeeper-3.4.9.jar:/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../src/java/lib/*.jar:/Users/zhangbingbing/Work/zookeeper-3.4.9/bin/../conf:
java.library.path=/Users/zhangbingbing/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.
java.io.tmpdir=/var/folders/0q/6th65vd574gfgtkn_800fs600000gn/T/
java.compiler=<NA>
os.name=Mac OS X
os.arch=x86_64
os.version=10.12.3
user.name=zhangbingbing
user.home=/Users/zhangbingbing
user.dir=/Users/zhangbingbing/Work/zookeeper-3.4.9

```

### ruok

> 用于输出当前zookeeper服务器是否正在运行

### stat

> 用于获取服务器的运行状态信息，包括基本的zookeeper版本、打包信息、运行时角色、集群数据节点个数等信息，另外还会将当前服务器的客户端连接信息打印出来

### srvr

> 和stat功能一致，唯一的区别是srvr不会将客户端的连接情况输出，仅仅输出服务器的自身信息

```
srvr
Zookeeper version: 3.4.9-1757313, built on 08/23/2016 06:50 GMT
Latency min/avg/max: 0/0/99
Received: 71422
Sent: 71647
Connections: 1
Outstanding: 0
Zxid: 0x28e
Mode: standalone
Node count: 7

```

### srst

> 用于重置所有服务器的统计信息

### wchs

> 用于输出当前服务器上管理的watcher的概要信息

```
wchs
0 connections watching 0 paths
Total watches:0
```

### wchc

> 用于输出当前服务器上管理的watcher的详细信息，以会话为单位进行归组，同时列出被该会话注册了watcher的节点路径


### wchp

> 用于输出当前服务器上管理的watcher的详细信息，不同点在于wchp命令的输出信息以节点路径为单位进行归组

### mntr

> 用于输出比stat命令更为详尽的服务器统计信息，包括请求处理的延迟情况，服务器内存数据库大小和集群的数据同步情况

```
zk_version      3.4.9-1757313, built on 08/23/2016 06:50 GMT
zk_avg_latency  0
zk_max_latency  0
zk_min_latency  0
zk_packets_received     4
zk_packets_sent 4
zk_num_alive_connections        1
zk_outstanding_requests 0
zk_server_state standalone
zk_znode_count  7
zk_watch_count  0
zk_ephemerals_count     0
zk_approximate_data_size        91
zk_open_file_descriptor_count   31
zk_max_file_descriptor_count    10240
```
