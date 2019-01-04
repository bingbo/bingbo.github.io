memcached默认采用的是Slab Allocator的机制分配管理内存的，在此之前，内存的分配是通过对所有的记录简单地进行malloc和free来进行的，但这种方式容易造成很多内存碎片，加重操作系统内存管理的负担。

## slab allocator机制

slab allocator的原理是将分配的内存分隔成各种尺寸的块，并把尺寸相同的块分成组，所分配的这些内存不会释放，而是重复利用.

针对客户端发送的数据，memcached会根据收到数据的大小，选择最适合数据大小的slab。memcached中保存着slab内空闲块的列表，根据该列表选择块，然后将数据组上于其中

## slab allocator缺点

该机制来带来了一定的问题，就是由于分配的是特定长度的内存，因此无法有效利用分配的内存，如，将100字节的数据缓存到128字节的块中，而剩余的28字节就浪费了

## 使用growth factor进行调优

memcached启动时可以指定growth factor因子，通过-f选项，就可以在某种程度上控制slab之间的差异，默认为1.25。可由`./memcached -f 2 -vv`查看内存分配情况

## 查看memcached内部状态

memcached有个stats的命令，它可以获得各种各样的信息，如：

```bash
$telnet localhost 11211
Trying ::1...
Connected to localhost.
Escape character is '^]'.
stats
STAT pid 4422
STAT uptime 85586
STAT time 1459390330
STAT version 1.4.25
STAT libevent 2.0.22-stable
STAT pointer_size 64
STAT rusage_user 0.329239
STAT rusage_system 0.546175
STAT curr_connections 10
....
```

通过`stats slabs`或`stats items`还可以获得关于缓存记录的信息。

## memcached的删除数据机制

数据不会从memcached中消失，不会释放已分配的内存，记录超时后，客户端就无法再看到该记录，其存储空间是可以再次利用

__lazy expiration__

memcached内部不会监视记录是否过期，而是在get时查看记录的时间戳，检查记录是否过期，这种技术被称之为lazy expiration，因此memcached不会在过期监视上耗费CPU时间

__LRU__

memcached会优先使用已超时的记录空间，但有时候也会发生空间不足的情况，这时间就会使用LRU机制来分配空间，就是说删除最近最少使用的记录来存放新的记录。

但可通过`./memcached -M`命令中的-M参数来禁止使用LRU机制，如果内存用尽时就会返回错误。
