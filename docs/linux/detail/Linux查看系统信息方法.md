### linux CPU大小

```bash
[root@idc ~]# cat /proc/cpuinfo |grep "model name" && cat /proc/cpuinfo |grep "physical id"
model name: Intel(R) Xeon(TM) CPU 2.80GHz
model name: Intel(R) Xeon(TM) CPU 2.80GHz
model name: Intel(R) Xeon(TM) CPU 2.80GHz
model name: Intel(R) Xeon(TM) CPU 2.80GHz
physical id    : 0
physical id    : 0
physical id    : 3
physical id    : 3
[root@idc ~]#
```

说明：Linux下可以在/proc/cpuinfo中看到每个cpu的详细信息。但是对于双核的cpu，在cpuinfo中会看到两个cpu。常常会让人误以为是两个单核的cpu。

其实应该通过Physical Processor ID来区分单核和双核。而Physical Processor ID可以从cpuinfo或者dmesg中找到. flags 如果有 ht 说明支持超线程技术 判断物理CPU的个数可以查看physical id 的值，相同则为同一个物理CPU

可以看到上面，这台机器有两个双核的CPU，ID分别是0和3，大小是2.8G。

### 内存大小

```bash
[root@xbidc ~]# cat /proc/meminfo |grep MemTotal
MemTotal:              1034612 kB
[root@xbidc ~]# 
```

### 硬盘大小

```bash
[root@xbidc ~]# fdisk -l |grep Disk
Disk /dev/sda: 300.0 GB, 300000000000 bytes
[root@xbidc ~]#
```

### 更多查看linux硬件信息的方法

```bash
uname -a               # 查看内核/操作系统/CPU信息的linux系统信息命令
head -n 1 /etc/issue   # 查看操作系统版本，是数字1不是字母L
cat /proc/cpuinfo      # 查看CPU信息的linux系统信息命令
hostname               # 查看计算机名的linux系统信息命令
lspci -tv              # 列出所有PCI设备
lsusb -tv              # 列出所有USB设备的linux系统信息命令
lsmod                  # 列出加载的内核模块
env                    # 查看环境变量资源
free -m                # 查看内存使用量和交换区使用量
df -h                  # 查看各分区使用情况
du -sh         # 查看指定目录的大小
grep MemTotal /proc/meminfo   # 查看内存总量
grep MemFree /proc/meminfo    # 查看空闲内存量
uptime                 # 查看系统运行时间、用户数、负载
cat /proc/loadavg      # 查看系统负载磁盘和分区
mount | column -t      # 查看挂接的分区状态
fdisk -l               # 查看所有分区
swapon -s              # 查看所有交换分区
hdparm -i /dev/hda     # 查看磁盘参数(仅适用于IDE设备)
dmesg | grep IDE       # 查看启动时IDE设备检测状况网络
ifconfig               # 查看所有网络接口的属性
iptables -L            # 查看防火墙设置
route -n               # 查看路由表
netstat -lntp          # 查看所有监听端口
netstat -antp          # 查看所有已经建立的连接
netstat -s             # 查看网络统计信息进程
ps -ef                 # 查看所有进程
top                    # 实时显示进程状态用户
w                      # 查看活动用户
id             # 查看指定用户信息
last                   # 查看用户登录日志
cut -d: -f1 /etc/passwd   # 查看系统所有用户
cut -d: -f1 /etc/group    # 查看系统所有组
crontab -l             # 查看当前用户的计划任务服务
chkconfig –list       # 列出所有系统服务
chkconfig –list | grep on    # 列出所有启动的系统服务程序
rpm -qa                # 查看所有安装的软件包
cat /proc/cpuinfo ：查看CPU相关参数的linux系统命令
cat /proc/partitions ：查看linux硬盘和分区信息的系统信息命令
cat /proc/meminfo ：查看linux系统内存信息的linux系统命令
cat /proc/version ：查看版本，类似uname -r
cat /proc/ioports ：查看设备io端口
cat /proc/interrupts ：查看中断
cat /proc/pci ：查看pci设备的信息
cat /proc/swaps ：查看所有swap分区的信息
```