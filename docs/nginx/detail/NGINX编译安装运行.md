## 安装 
首先安装相应的依赖，如PCRE、zlib、OpenSSL相应的lib等，然后再进行nginx相应的安装   
[参考文档](https://www.nginx.com/resources/admin-guide/installing-nginx-open-source/)
### PCRE library
> 为nginx的核心模块和Rewrite模块提供正则表达式支持

```nginx
#wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.tar.gz
wget http://tenet.dl.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz
tar -zxf pcre-8.37.tar.gz
cd pcre-8.37
./configure
make
sudo make install
```

### zlib library
> 为nginx的Gzip模块提供headers头压缩功能。

```nginx
wget http://zlib.net/zlib-1.2.8.tar.gz
tar -zxf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure
make
sudo make install
```
### OpenSSL library
> 为nginx的SSL模块提供https协议支持。

```nginx
wget http://www.openssl.org/source/openssl-1.0.2d.tar.gz
tar -zxf openssl-1.0.2d.tar.gz
cd openssl-1.0.2d
./configure darwin64-x86_64-cc --prefix=/usr
make
sudo make install
```
### nginx source
> 下载并结合相应的依赖包进行nginx安装

```nginx
wget http://nginx.org/download/nginx-1.9.5.tar.gz
tar zxf nginx-1.9.5.tar.gz
cd nginx-1.9.5
```
```nginx
./configure 
    --conf-path=/home/users/zhangbingbing/nginx/conf/nginx.conf 
    --sbin-path=/home/users/zhangbingbing/nginx/sbin/nginx 
    --pid-path=/home/users/zhangbingbing/nginx/logs/nginx.pid 
    --with-http_ssl_module 
    --with-pcre=vendor/pcre-8.37/ 
    --with-zlib=vendor/zlib-1.2.8/ 
    --prefix=/home/users/zhangbingbing/nginx 
    --with-http_realip_module
make && make install
```

***

## 进程及运行控制

### 主进程与工作进程

> nginx有一个主进程与一个或多个工作进程，如果缓存启用的话，缓存加载器与管理器也会在启动的时候运行。  
主进程的主要目的是读取各检查配置文件，同时也维护工作进程。工作进程则是实际处理请求，nginx根据操作系统的机制有效地分发请求到各个工作进程。

### 控制nginx 

> 为了重新加载配置文件，你可以停止或重启nginx，或是发送信号给主进程master。

```nginx
nginx -s signal
```

*signal信号的值可以是以下几个：*
* quit -优雅的关闭
* reload -重新加载配置文件
* reopen -重新打开日志文件
* stop -立即快速关闭

