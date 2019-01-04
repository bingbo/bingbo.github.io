## 下载nginx源码并安装

```bash
/Users/zhangbingbing/Work/nginx_dev/nginx-1.9.5
#简单安装并开启调试模式
./configure --prefix=/Users/zhangbingbing/Work/nginx_dev --conf-path=/Users/zhangbingbing/Work/nginx_dev/conf/nginx.conf --sbin-path=/Users/zhangbingbing/Work/nginx_dev/sbin/nginx --pid-path=/Users/zhangbingbing/Work/nginx_dev/logs/nginx.pid --with-debug
make && make install
```

> 配置nginx.conf只开启一个worker进程，`worker_processes 1`,`daemon on;`和`master_process on;`默认开启

## gdb调试

```bash
#q选项是以安静模式启动，不显示GDB版本等信息。tui选项可以显示代码界面
sudo gdb -q -tui
#在gdb里启动nginx
(gdb) shell ./nginx
#查看nginx的Worker进程id
(gdb) shell ps -ef | grep nginx
#用attach命令来跟踪子进程
(gdb) attach 7367
#设置断点
(gdb) b ngx_process_events_and_timers 
#然后采用命令c，使得nginx一直运行，直到遇到第一个断点
(gdb) c
```

接下来就可以发个请求调试了，其中`n,s`都可以一步步的执行，但`s`可以进入每个函数内