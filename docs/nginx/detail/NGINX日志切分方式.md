## 定时任务切分

所谓的定时任务切分，是指通过定时任务（比如crontab)，发送信号给nginx，让其重新打开文件。

发送USR1 信号会让nginx主动重新打开日志文件：

```bash
$ mv access.log access.log.0
$ kill -USR1 `cat master.nginx.pid`
$ sleep 1
$ gzip access.log.0    # do something with access.log.0
```

> 优点是思路较为简单，但效果明显，而且对error_log 同样适用；缺点是有外部依赖（比如 crontab)

## 自切分

自切分是指让nginx自身实现日志切分功能，不依赖crontab等东西。 其主要原理是依赖access_log的强大功能---- 可以用变量定义请求的log路径。

但每打印一个请求log就得打开文件，写日志，关闭文件，性能效率比较差，所以借助于`open_file_cache`指令，即存打开的文件，只有满足一定条件的时候才会重新去check当前fd对应的文件是否合法，是否需要重新打开。

```nginx
Syntax:     open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time];
            open_log_file_cache off;
Default:     open_log_file_cache off;
Context:     http, server, location
```

* max : 设置缓存中描述符的最大数量；如果缓存被占满，最近最少使用（LRU）的描述符将被关闭
* inactive : 设置缓存文件描述符在多长时间内没有被访问就关闭； 默认为10秒
* min_uses : 设置在inactive参数指定的时间里， 最少访问多少次才能使文件描述符保留在缓存中；默认为1
* valid :设置一段用于检查超时后文件是否仍以同样名字存在的时间； 默认为60秒
* off :禁用缓存

综上，要让nginx自切分，需要两个步骤，其一，配置合理的access_log;其二，开启open_log_file_cache提升性能:

```nginx
#提取nginx变量
if ($time_iso8601 ~ "^(\d{4})-(\d{2})-(\d{2})T(\d{2}):(\d{2}):(\d{2})"){
    set $year $1;
    set $month $2;
    set $day $3;
    set $hour $4;
    set $minutes $5;
    set $seconds $6;
}


#配置access_log ；以  hour 为界
access_log  logs/access.log.$year$month$day$hour;

#配置open_log_file_cache
open_log_file_cache max=10 inactive=60s valid=1m min_uses=2;
```

> 总结 : 自切分可一定程度上面满足日志切分的需求；但是对性能会有一定的影响； 另外，并不支持error_log的切分
