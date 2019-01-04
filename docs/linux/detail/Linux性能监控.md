系统性能优化是一项复杂的工作，根据监测、测试、评估等结果，找出系统的瓶颈，危地马拉不同的系统、不同的硬件、不同的应用进行重点优化。由于系统是由若干子系统构成的，修改一个地方可能会影响另一个地方，所以性能优化是优化、测试、监控等连在一起的，是一个长期的过程。

一般监测性能会从CPU、Memory、IO、Network等几个方面入手。

## 查看系统信息

* 查看CPU相关信息 

    ```bash
    $ cat /proc/cpuinfo
    processor   : 0
    vendor_id   : GenuineIntel
    cpu family  : 6
    model       : 42
    model name  : Intel Xeon E312xx (Sandy Bridge)
    stepping    : 1
    cpu MHz     : 2000.000
    cache size  : 4096 KB
    fpu     : yes
    fpu_exception   : yes
    cpuid level : 13
    wp      : yes
    ```

* 查看中断相关信息 

    ```bash
    $cat /proc/interrupts  
               CPU0       CPU1
    0:        135          1   IO-APIC-edge      timer
    1:          0          6   IO-APIC-edge      i8042
    4:          3          2   IO-APIC-edge      serial
    6:          1          1   IO-APIC-edge      floppy
    9:          0          0   IO-APIC-fasteoi   acpi
    11:    7194337    7386529   IO-APIC-fasteoi   uhci_hcd:usb1, virtio5
    ...
    ```

* 查看设备IO端口 

    ```bash
    $cat /proc/ioports  
    0000-001f : dma1
    0020-0021 : pic1
    0040-0043 : timer0
    0050-0053 : timer1
    0060-0060 : keyboard
    0064-0064 : keyboard
    0070-0077 : rtc
    0080-008f : dma page reg
    ...
    ```

* 查看内存信息 

    ```bash
    $cat /proc/meminfo  
    MemTotal:        4059860 kB
    MemFree:          147424 kB
    Buffers:          223912 kB
    Cached:          2425420 kB
    SwapCached:        18324 kB
    Active:          2346936 kB
    Inactive:        1100484 kB
    ...
    ```

* 查看所有设备的所有分区 

    ```bash
    $cat /proc/partitions  
    major minor  #blocks  name

    253        0   20971520 vda
    253        1   20969636 vda1
    253       16   41943040 vdb
    253       32    4194304 vdc
    ```

* 查看所有Swap分区的信息 
    
    ```bash
    $cat /proc/swaps  
    Filename                Type        Size    Used    Priority
    /dev/vdc                                partition   4194296 247708  -1
    ```

* 查看Linux的版本号 相当于 uname -r 

    ```bash
    $cat /proc/version  
    Linux version 2.6.32_1-16-0-0_virtio (root@xxx.com) (gcc version 4.4.6 20120305 (Red Hat 4.4.6-4) (GCC) ) #1 SMP Thu May 14 15:30:56 CST 2015
    ```
    
* 查看处理器使用率

    ```bash
    $cat /proc/stat
    cpu  46247529 36076667 110691128 2746662766 1076846 75035 2130412 8680773 0
    cpu0 28512296 7476882 45817969 1388191305 610871 37136 1191439 3875637 0
    cpu1 17735232 28599784 64873159 1358471461 465974 37898 938973 4805135 0
    ...
    ```
* 查看网络利用率

    ```bash
    $cat /proc/net/dev
    Inter-|   Receive                                                |  Transmit
    face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
    lo:248619887796 2493175801    0    0    0     0          0         0 248619887796 2493175801    0    0    0     0       0          0
    eth0:86895313484 314630906    0    0    0     0          0         0 67240165736 331657222    0    0    0     0       0          0
    ```

* 看系统内核等信息

    ```bash
    $uname -a
    Linux xxx.yyy.com 2.6.32_1-16-0-0_virtio #1 SMP Thu May 14 15:30:56 CST 2015 x86_64 x86_64 x86_64 GNU/Linux
    ```

## 性能监控工具

### CPU监控

常用的监视工具有mpstat,vmstat,top.

__mpstat__

它是一个实时系统监控工具。其报告与CPU的一些统计信息，这些信息放在/proc/stat文件中。在多CPU系统里，不但能看到所有CPU信息，还能查看特定CPU信息.

```bash
mpstat  [  -A  ] [ -I { SUM | CPU | SCPU | ALL } ] [ -u ] [ -P { cpu [,...] | ALL } ] [ -V ] [ interval [count ] ]
```

__参数说明__

* -A 相当于-I ALL -u -P ALL等参数组合
* -P {cpu|ALL} 表示监控哪个CPU，cpu在【0 - num-1】中取值
* -V 显示版本号并退出
* interval 相邻两次采样的时间间隔
* count 采样的次数，count只能和delay一起使用

当没有参数时，mpstat则显示系统启动以后所有信息的平均值。有interval时，第一行的信息自系统启动以来平均值。从第二行开始，输出为前一个interval时间段的平均信息，如下所示

```bash
$ mpstat -P ALL 1 2
Linux 2.6.32_1-16-0-0_virtio (ibingbo.com)    05/03/2016  _x86_64_    (2 CPU)

11:58:55 AM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
11:58:56 AM  all    1.50    1.50    2.00    0.00    0.00    0.00    0.00    0.00   95.00
11:58:56 AM    0    2.02    1.01    2.02    0.00    0.00    0.00    1.01    0.00   93.94
11:58:56 AM    1    0.00    2.00    2.00    0.00    0.00    0.00    0.00    0.00   96.00

11:58:56 AM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
11:58:57 AM  all    1.01    2.01    4.52    0.00    0.00    0.00    0.50    0.00   91.96
11:58:57 AM    0    1.98    0.99    3.96    0.00    0.00    0.00    0.00    0.00   93.07
11:58:57 AM    1    1.00    4.00    6.00    0.00    0.00    0.00    0.00    0.00   89.00

Average:     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
Average:     all    1.25    1.75    3.26    0.00    0.00    0.00    0.25    0.00   93.48
Average:       0    2.00    1.00    3.00    0.00    0.00    0.00    0.50    0.00   93.50
Average:       1    0.50    3.00    4.00    0.00    0.00    0.00    0.00    0.00   92.50
```

参数说明：

* cpu 处理器ID
* user 在interval时间段里，用户态的cpu时间，不包含nice值为负进程
* nice 在interval时间里，nice值为父进程的cpu时间
* sys 在interval时间里，核心时间
* iowait 在interval时间里，IO等待时间
* irq 在interval时间里，硬中断时间
* soft 在interval时间里，软中断时间
* idle 在interval时间里，除去等待IO操作外的闲置时间

__vmstat__

vmstat是Virtual Memory Statistics（虚拟内存统计）的缩写，是实时系统监控工具。该工具反应了与CPU的相应信息，包括多少任务在运行、CPU使用的情况、CPU收到多少中断、发生多少上下文切换等

```bash
vmstat [-a] [-n] [delay [ count]]
vmstat [-f] [-s] [-m]
vmstat [-S unit]
vmstat [-d]
vmstat [-p disk partition]
vmstat [-V]
```

参数说明：

* -a 打开显示活动与非活动的内存情况
* -f 显示由启动进程fork的进程数量
* -m 显示slab信息
* -s 以表格的形式显示
* -S 以k,m为单位来输出显示 
* -V 显示版本号
* -d 显示磁盘统计
* delay 相邻两次采样的时间间隔
* count 采样的次数

当没有参数时，mvstat则显示系统启动以后所有的平均值。有delay时，第一行是系统启动以来的平均信息。从第二行开始输出前一个delay时间段的平均信息。当有多个CPU时，输出为所有CPU的平均值


```bash
$ vmstat
procs -----------memory---------- ---swap-- -----io---- --system-- ----cpu----
r  b   swpd   free   buff  cache   si   so    bi    bo   in    cs us sy id wa
2  0 245976 123552 245576 2414688    0    0     0    33    1     1  3  4 93  0
```

说明：

* r 等待运行的任务数
* b 阻塞的任务数
* swpd 已使用的虚拟内存的量
* free 空闲内存的量
* buff 作为缓冲区的使用量
* cache 作为缓存的使用量
* inact 为激活的内存量
* active 已激活的内存量
* si 从磁盘换入内存的量
* so 换出内存的量
* in 每秒中断数
* cs 每秒上下文切换数
* us 用户态的CPU时间
* sy 内核态的CPU时间
* id 空闲时间
* wa 等待IO时间

__top__

top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况。

它是一个动态显示过程，即可以通过用户按键来不断刷新当前状态。如果在前台执行该命令，则它将独占前台，直到用户终止该程序为止。也就是说top提供了实时的对系统处理器的状态监视，它将显示系统中CPU最"敏感"的任务列表，可以按CPU使用、内存使用和执行时间对任务进行排序，且很多特性都可以通过交互式命令或者在个人定制文件中进行设定。

```bash
top [-] [d] [p] [q] [c] [C] [S]    [n]
```

参数说明：

* d  指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之。
* p  通过指定监控进程ID来仅仅监控某个进程的状态。
* q 该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。
* S 指定累计模式
* s  使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。
* i  使top不显示任何闲置或者僵死进程。
* c  显示整个命令行而不只是显示命令名

```bash
top - 09:47:03 up 180 days, 16:57,  3 users,  load average: 0.01, 0.02, 0.02
Tasks: 548 total,   1 running, 546 sleeping,   1 stopped,   0 zombie
Cpu(s):  1.5% us,  3.3% sy,  0.7% ni, 94.5% id,  0.0% wa,  0.0% hi,  0.0% si
Mem:   4059860k total,  3935868k used,   123992k free,   213420k buffers
Swap:  4194296k total,   278328k used,  3915968k free,  2446008k cached

PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
31163 root      39  19  504m  48m  17m S  4.6  1.2 308:59.68 baas_agent
8044 work      20   0  6612 1356  760 R  1.0  0.0   0:07.30 top
18726 root      39  19  261m  90m 3636 S  0.7  2.3 301:47.66 naming-agent
```

前5行为系统整体统计信息

第一行为任务队列信息，同uptime命令结果一样，如：

```bash
$ uptime
09:46:23 up 180 days, 16:56,  3 users,  load average: 0.02, 0.03, 0.03
```

* _09:46:23_ 当前时间
* _up 180 days, 16:56_ 系统运行时间
* _3 users_ 当前登录用户数
* _load average_ 系统负载，即任务队列的平均长度。三个数值分别为1分钟、5分钟、15分钟前到现在的平均值

第二、三行为进程和CPU的信息，当有多个CPU时，这些内容可能会超过两行

* _548 total_ 表示进程总数
* _1 running_ 正在运行的进程数
* _546 sleeping_ 睡眠的进程数
* _1 stopped_ 停止的进程数
* _0 zombie_ 僵尸进程数
* _Cpu(s):  1.5% us_ 用户空间占用CPU百分比
* _3.3% sy_ 内核空间占用CPU百分比
* _0.7% ni_ 用户进程空间内改变过优先级的进程占用CPU百分比
* _94.5% id_ 空闲CPU百分比 
* _0.0% wa_ 等待输入输出的CPU百分比

第四、五行为内存信息

* _Mem:   4059860k total_ 物理内存总量 
* _3935868k used_ 使用的物理内存总量
* _123992k free_ 空闲内存总量
* _213420k buffers_ 用作内核缓存的内存量
* _Swap:  4194296k total_ 交换区总量
* _278328k used_  交换区总量
* _3915968k free_  空闲交换区总量
* _2446008k cached_ 缓冲的交换区总量。 内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖， 该数值即为这些内容已存在于内存中的交换区的大小。相应的内存再次被换出时可不必再对交换区写入。

其他为具体的进程信息

* PID 进程id
* PPID 父进程id
* RUSER Real user name
* UID 进程所有者的用户id
* USER 进程所有者的用户名
* GROUP 进程所有者的组名
* TTY 启动进程的终端名。不是从终端启动的进程则显示为 ?
* PR 优先级
* NI nice值。负值表示高优先级，正值表示低优先级
* P 最后使用的CPU，仅在多CPU环境下有意义
* %CPU 上次更新到现在的CPU时间占用百分比
* TIME 进程使用的CPU时间总计，单位秒
* TIME+ 进程使用的CPU时间总计，单位1/100秒
* %MEM 进程使用的物理内存百分比
* VIRT 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
* SWAP 进程使用的虚拟内存中，被换出的大小，单位kb。
* RES 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
* CODE 可执行代码占用的物理内存大小，单位kb
* DATA 可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
* SHR 共享内存大小，单位kb
* nFLT 页面错误次数
* nDRT 最后一次写入到现在，被修改过的页面数。
* S 进程状态。

    * D=不可中断的睡眠状态
    * R=运行
    * S=睡眠
    * T=跟踪/停止
    * Z=僵尸进程

* COMMAND 命令名/命令行
* WCHAN 若该进程在睡眠，则显示睡眠中的系统函数名
* Flags 任务标志，参考 sched.h

默认只显示一部分列信息，改变显示内容可以通过f键选择，o键可以改变列的显示顺序，按回车返回界面。


__iostat__

统计CPU和输入输出的统计信息,监控系统输入输出的负载情况

```bash
iostat [ -c ] [ -d ] [ -N ] [ -k | -m ] [ -t ] [ -V ] [ -x ] [ -z ] [ device [...] | ALL ] [ -p [  device[,...] | ALL ] ] [ interval [ count ] ]
```

选项说明：

* -c 显示CPU使用情况
* -d 显示设备使用情况
* -k 以千字节每秒的形式显示统计数据
* -m 以兆字节每秒的形式显示统计数据
* -p [{device|ALL}] 显示块设备或所有分区的使用情况
* interval 每隔多长时间输出一次
* count 总共输出多少次

```bash
$ iostat
Linux 2.6.32_1-16-0-0_virtio (ibingbo.com)    05/03/2016  _x86_64_    (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.56    1.22    3.82    0.04    0.30   93.07

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               2.00         0.23        45.64    3499521  690317268
vdb               2.40         0.56        19.48    8396450  294591184
vdc               0.00         0.01         0.06     135612     900288
```

数据说明：

* user 用户态的CPU时间
* nice 用户态时设置了nice值进程的CPU使用时间
* system 内核态CPU时间
* iowait 硬盘IO等待时间
* steal 
* idle CPU空闲时间
* tps 设备每秒传输速度
* blk_read/s 每秒从设备读的数据量
* blk_wrtn/s 每秒写设备的数据量

### MEMORY内存监控

内存有虚拟内存和物理内存，虚拟内存是采用硬盘来对物理内存进行扩展，将暂时不用的内存页写到硬盘上让出更多的物理内存空间。当需要的时候再从硬盘读回内存。这些对用户来说是透明的。Linux下的swap分区就是虚拟内存。

每个进程启动时都会向系统申请虚拟内存(VSZ)，同时在使用时会映射到物理内存上，而RSS表示程序所占物理内存的大小。

__PS查看进程占用内存情况__

可以看到进程占用的VSZ和RSS，如下所示：

```bash
$ ps aux
USER       PID %CPU %MEM   VSZ  RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  4828  456 ?        Ss    2015   3:06 init [3]
root         2  0.0  0.0     0    0 ?        S     2015   0:00 [kthreadd]
root         3  0.0  0.0     0    0 ?        S     2015   2:18 [migration/0]
root         4  0.0  0.0     0    0 ?        S     2015   6:01 [ksoftirqd/0]
root         5  0.0  0.0     0    0 ?        S     2015   0:00 [migration/0]
```

__free命令__

该命令用来监控内存使用情况

```bash
free [-b | -k | -m] [-o] [-s delay ] [-t] [-V]
```

选项说明：

* -b|-k|-m 分别以字节、千字节、兆字节显示
* -t 多显示一行汇总统计
* -V 显示版本号
* -o 不显示buffers/cache使用情况
* -s delay 每多少秒显示一次

```bash
$ free
            total       used       free     shared    buffers     cached
Mem:       4059860    3938748     121112          0     219468    2446604
-/+ buffers/cache:    1272676    2787184
Swap:      4194296     247488    3946808
```

数据说明：

* mem 物理内存
* total 不显示核心使用的物理内存
* used 被使用的内存大小
* free 空间的内存大小
* shared 多个进程共享的内存大小
* buffers 显示磁盘缓存的当前大小
* swap 对换空间使用的大小

__iostat监控IO命令__

```bash
$ iostat  -x
Linux 2.6.32_1-16-0-0_virtio (ibingbo.com)    05/04/2016  _x86_64_    (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.56    1.22    3.82    0.04    0.30   93.07

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     9.57    0.01    1.99     0.23    45.64    45.78     0.00    0.87    2.18    0.87   0.46   0.09
vdb               0.02     2.52    0.05    2.35     0.55    19.48    16.70     0.01    2.16    1.05    2.19   1.40   0.34
vdc               0.00     0.01    0.00    0.00     0.01     0.06   150.51     0.00    2.72    3.14    2.41   2.15   0.00
```

数据说明：

* rrqm/s  每秒进行 merge 的读操作数目
* wrqm/s  每秒进行 merge 的写操作数目
* r/s 每秒完成的读 I/O 设备次数
* w/s 每秒完成的写 I/O 设备次数
* rsec/s  每秒读扇区数
* wsec/s  每秒写扇区数
* rkB/s   每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节
* wkB/s   每秒写K字节数。是 wsect/s 的一半
* avgrq-sz    平均每次设备I/O操作的数据大小 (扇区)
* avgqu-sz    平均I/O队列长度
* await   平均每次设备I/O操作的等待时间 (毫秒)
* svctm   平均每次设备I/O操作的服务时间 (毫秒)
* %util   一秒中有百分之多少的时间用于 I/O 操作，或者说一秒中有多少时间 I/O 队列是非空的

### NETWORK

影响网络的因素比较多，包括延迟、冲突、阻塞等

__iptraf__

iptraf是一个很好的查看本机网络吞吐量的好工具，支持文字图形界面

```bash
iptraf { [ -f ] [ -q ] [ { -i iface | -g | -d iface | -s iface | -z iface | -l iface } [ -t timeout ] [ -B [ -L logfile ] ] ] | [ -h ] }
```

选项说明：

* -i iface 查看指定网络接口上的网络情况
* -d iface 查看指定网络接口上的详细网络情况
* -s iface 查看指定网络接口上的所有的TCP和UDP传输情况
* -t timeout 告诉IPtraf在特定的设备上运行多少分钟

```bash
$iptraf -d eth0
```

![](https://github.com/bingbo/blog/blob/master/images/iptraf.png)

__tcpdump和tcptrace__

tcpdump和tcptrace提供了一种更细致的分析方法，先用tcpdump按要求获取数据包把结果输出到一文件，然后再用tcptrace体质其文件内容


```bash
tcpdump [-c count][-r file][-w file]
```
选项说明:

* -A 以ASCII形式打印每一个包
* -c 收到num数量的包后退出
* -d 获取匹配相应码的编译后的数据包并以易读的形式输出
* -r 从文件中读取包信息
* -w 输出数据包到指定的文件

```bash
$tcpdump -e -w net.dump
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes

$tcptrace network.dump
```



### 进程内存占用查看方法

1. top

    可以用top查看%MEM的内容，或按进程或用户查看，如
    ```bash
    top -u work`
    top - 11:11:57 up 175 days, 18:22,  2 users,  load average: 0.03, 0.04, 0.00
    Tasks: 541 total,   1 running, 539 sleeping,   1 stopped,   0 zombie
    Cpu(s):  1.0% us,  4.3% sy,  1.7% ni, 92.8% id,  0.0% wa,  0.0% hi,  0.2% si
     Mem:   4059860k total,  3984056k used,    75804k free,   220880k buffers
     Swap:  4194296k total,   247488k used,  3946808k free,  2485000k cached

    PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
    3904 work      20   0  6612 1348  760 R  0.7  0.0   0:00.32 top
    7004 work      20   0  245m  12m  632 S  0.3  0.3 123:07.27 php-cgi
    3667 work      20   0 29688 1824  648 S  0.0  0.0   0:00.00 nginx
    3668 work      20   0 35016 5740 1340 S  0.0  0.1   1:29.40 nginx
    3669 work      20   0 35016 6092 1676 S  0.0  0.2   0:00.00 nginx
    ```

2. pmap

    可以根据进程查看进程相关信息占用内存情况，如
    
    ```bash
    $ pmap -x 3667
    3667:   nginx: master process /home/work/odp_dev/webserver/sbin/nginx
    Address           Kbytes     RSS    Anon  Locked Mode   Mapping
    00000000003ff000       4       -       -       - rw---  nginx
    0000000000400000    1060       -       -       - r-x--  nginx
    0000000000608000     112       -       -       - rw---  nginx
    0000000000624000    4408       -       -       - rw---    [ anon ]
    000000302ad00000      84       -       -       - r-x--  ld-linux-x86-64.so.2
    000000302ae14000       8       -       -       - rw---  ld-linux-x86-64.so.2
    ```

3. ps

    显示work用户进程内存情况，并按内存由大到小排序

    ```bash
    $ ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid' | grep work | sort -nrk5
    6522 php-cgi          /home/work/odp_d  0.0 17980 255144 Jan15 work     502
    6516 php-cgi          /home/work/odp_d  0.0 17520 253872 Jan15 work     502
    6514 php-cgi          /home/work/odp_d  0.0 16600 255012 Jan15 work     502
    7003 ral-agent        /home/work/odp_d  0.1 16404 252372 2015 work      502
    6519 php-cgi          /home/work/odp_d  0.0 16092 255144 Jan15 work     502
    ```
