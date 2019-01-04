## 简介

memcached是高速运行的分布式缓存服务器，有以下特征：

* 协议简单

    > memcached的服务器与客户端通信并不使用复杂的xml等格式，而是使用了简单的基于文本行的协议。通过telnet也能在memcached上保存数据等。

* 基于libevent的事件处理

    > libevent是个程序库，它是将linux的epoll、BSD类操作系统的kqueue等事件处理功能封装成统一的接口。即使服务器的连接数增加，也能在Linux、BSD、Solaris等操作系统上发挥其高性能

* 内置内存存储方式

    > 为了提高性能，memcached 中保存的数据都存储在 memcached 内置的内存存储空间中。由于数据仅存在于内存中，因此重启 memcached、重启操作系统会导致全部数据消失。另外，内容容量达到指定值之后，就基于 LRU(Least Recently Used)算法自动删除不使用的缓存。memcached 本身是为缓存而设计的服务器，因此并没有过多考虑数据的永久性问题

* 分布式节点不相互通信

    > 虽说是分布式缓存服务器，但服务器端并没有分布式功能，各个节点不会相互通信以共享信息，该所谓的分布式完全取决于客户端的实现


## 安装运行

### 下载编译安装

```bash
wget http://memcached.org/latest
tar -zxvf memcached-1.x.x.tar.gz
cd memcached-1.x.x
./configure --prefix=/User/work/memcached
make && make install
```

### 运行

```bash
./memcached -p 11211 -l 127.0.0.1 -d -m 64m
```

_options:_

* -p    指定端口，默认为11211
* -l    指定host
* -d    指定作为后台程序运行
* -m    指定分配的内存大小，默认为64m

### brew安装

```bash
brew install memcached
```
> 启动

```bash
brew services start memcached
#或
/usr/local/opt/memcached/bin/memcached
```

> 连接进行读写操作

```
$telnet localhost 11211
Trying ::1...
Connected to localhost.
Escape character is '^]'.
set foo 0 0 3
bar
STORED
get foo
VALUE foo 0 3
bar
END
```
