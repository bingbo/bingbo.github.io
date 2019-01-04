ab是Apache超文本传输协议(HTTP)的性能测试工具。其设计意图是描绘当前所安装的Apache的执行性能，主要是显示你安装的Apache每秒可以处理多少个请求。

`ab -n 1000 -kc 1000 http://localhost:8080/`

其中－n表示请求数，－c表示并发数, -k 使用http的keepalive特性

> 可看如下结果

```nginx
$ab -kc 1000 -n 10000 http://1localhost:8080/
This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.218.138.6 (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        nginx
Server Hostname:        localhost
Server Port:            8080

Document Path:          /
Document Length:        537 bytes

Concurrency Level:      1000
Time taken for tests:   6.379 seconds
Complete requests:      10000
Failed requests:        111
   (Connect: 0, Receive: 0, Length: 111, Exceptions: 0)
Non-2xx responses:      10000
Keep-Alive requests:    9889
Total transferred:      7395267 bytes
HTML transferred:       5310393 bytes
Requests per second:    1567.54 [#/sec] (mean)
Time per request:       637.942 [ms] (mean)
Time per request:       0.638 [ms] (mean, across all concurrent requests)
Transfer rate:          1132.07 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    6  20.3      0     104
Processing:    47  136 387.1     98    5730
Waiting:       46  136 387.1     98    5730
Total:         63  142 394.0    100    5818

Percentage of the requests served within a certain time (ms)
  50%    100
  66%    106
  75%    110
  80%    113
  90%    134
  95%    169
  98%    176
  99%   1181
 100%   5818 (longest request)
```

> 结果说明

```nginx
Requests per second:    6065.46 [#/sec] (mean)
Time per request:       164.868 [ms] (mean)
Time per request:       0.165 [ms] (mean, across all concurrent requests)

Requests per second
#表示当前测试的服务器每秒可以处理6065.46个静态html的请求事务，后面的mean表示平均。这个数值表示当前机器的整体性能，值越大越好。

Time per request
#单个并发的延迟时间，后面的mean表示平均。
#隔离开当前并发，单独完成一个请求需要的平均时间。

##################################################

说一下两个Time per request区别

Time per request:       164.868 [ms] (mean)
Time per request:       0.165 [ms] (mean, across all concurrent requests)
 
#前一个衡量单个请求的延迟，cpu是分时间片轮流执行请求的，多并发的情况下，一个并发上的请求时需要等待这么长时间才能得到下一个时间片。

计算方法Time per request:       0.165 [ms] (mean, across all concurrent requests)*并发数

#通俗点说就是当以-c 10的并发下完成-n 1000个请求的同时，额外加入一个请求，完成这个求平均需要的时间。
#后一个衡量性能的标准，它反映了完成一个请求需要的平均时间,在当前的并发情况下，增加一个请求需要的时间。

计算方法Time taken for tests: 60.444 seconds/Complete requests: 1000

#通俗点说就是当以-c 10的并发下完成-n 1001个请求时，比完成-n1000个请求多花的时间。
#你可以适当调节-c 和-n大小来测试服务器性能，借助htop指令来直观的查看机器的负载情况。
```