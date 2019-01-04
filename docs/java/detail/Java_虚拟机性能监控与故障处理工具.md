### jps-虚拟机进程状况工具

#### 命令格式

可以列出正在运行的虚拟机进程，并显示虚拟机执行主类(Main class,main()函数所在的类)名称以及这些进程的本地虚拟唯一ID

```
jps [options] [hostid]
```

```
$jps -l
27316 org.jetbrains.jps.cmdline.Launcher
33303 org.apache.catalina.startup.Bootstrap
25611 com.intellij.database.remote.RemoteJdbcServer
37564 sun.tools.jps.Jps
```

#### 常用选项

|选项|作用
|---|---
|-q|只输出LVMID，省略主类的名称
|-m|输出虚拟机进程启动时传递给主类main()函数的参数
|-l|输出主类的全名，如果 进程执行的是Jar包，输出Jar路径
|-v|输出虚拟机进程忘却时JVM参数

### jstat-虚拟机统计信息监视工具

jstat(JVM Statistics Monitoring Tool)是用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据

#### 命令格式

```
jstat [option vmid ]interval[s|ms] [count]] ]
```

```
#每250毫秒查询一次，共查20次
jstat -gc 2764 250 20
S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
86016.0 5120.0  0.0   4688.3 481792.0 186967.7  223232.0   112891.2  57176.0 56069.4 6528.0 6198.2     27    1.152   4      0.510    1.662
86016.0 5120.0  0.0   4688.3 481792.0 186967.7  223232.0   112891.2  57176.0 56069.4 6528.0 6198.2     27    1.152   4      0.510    1.662
86016.0 5120.0  0.0   4688.3 481792.0 186967.7  223232.0   112891.2  57176.0 56069.4 6528.0 6198.2     27    1.152   4      0.510    1.662
86016.0 5120.0  0.0   4688.3 481792.0 186967.7  223232.0   112891.2  57176.0 56069.4 6528.0 6198.2     27    1.152   4      0.510    1.662
86016.0 5120.0  0.0   4688.3 481792.0 186967.7  223232.0   112891.2  57176.0 56069.4 6528.0 6198.2     27    1.152   4      0.510    1.662
86016.0 5120.0  0.0   4688.3 481792.0 186967.7  223232.0   112891.2  57176.0 56069.4 6528.0 6198.2     27    1.152   4      0.510    1.662
```

* E-新生代Eden区
* S0,S1-表示Survivor0,Survivor1区
* O-old，老年代
* P-Permanent,永久代
* YGC-表示Young GC|Minor GC
* FGC-Full GC
* FGCT-Full GC TIme总耗时
* GCT-GC Time

#### 选项参数

|选项|作用|
|--|--|
|-class|监视类装载、卸载数量、总空间以及类半截所耗费的时间
|-gc|监视Java堆状况，包括Eden区、两个survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息
|-gccapacity|监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间
|-gcutil|监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比
|-gccause|与-gcutil功能一样，但是会额外输出导致上一次GC产生的原因
|-gcnewcapacity|监视内容与-gcnew基本相同，全国联网 主要关注使用到的最大、最小空间
|-gcnew|监视新生代GC状况
|-gcold|监视老年代GC状况
|-gcoldcapacity|监视内容与-gcold基本相同，全国联网 主要关注使用到的最大、最小空间
|-gcpermcapacity|输出永久代使用到的最大、最小空间
|-compiler|输出JIT编译器编译过的方法、耗时等信息
|-printcompilation|输出已经被JIT编译的方法


### jinfo-Java配置信息工具

实时地查看和调整虚拟机各项参数，使用jps命令的-v参数 可以查看虚拟机启动时显式指定的参数列表，但如果想知道未被显式指定的参数的系统默认值，就只能使用jinfo的-flag选项了

```
jinfo [option] pid
```

### jmap-java内存映像工具

jmap(Memory Map for Java)命令用于生成堆转储快照，还可以查询finalize执行队列、Java堆和永久代的详细信息，如空间使用率、当前用的哪种收集器等

```
jmap [option] vmid
```

|选项|作用|
|---|---|
|-dump|生成java堆转储快照。格式：-dump:[live,]format=b,file=\<filename\>,其中live子参数说明是否只dump出存活的对象
|-finalizerinfo|显示在F-Queue中等待Finalizer线程执行finalize方法的对象。只在Linux/Solaris平台下有效
|-heap|显示Java堆详细信息，如使用哪种回收器、参数配置、分代状况等，只在Linux/Solaris平台下有效
|-histo|显示堆中对象统计信息，包括类，实例数量，合计容量
|-permstat|以ClassLoader为统计口径显示永久代内存状态，只在Linux/Solaris平台下有效
|-F|当虚拟机进程对-dump选项没有响应时，可使用这个选项强制生成dump快照，只在linux/solaris平台下有效

### jhat-虚拟机堆转储快照分析工具

与jmap搭配酸胀，来分析jmap生成的堆转储快照

### jstack-java堆栈跟踪工具

用于生成虚拟机当前时间的线程快照，线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的常见原因

```
jstack [option] vmid
```

|选项|作用|
|-F|当正常输出的请求不被响应时，强制输出线程堆栈
|-l|除堆栈外，显示关于锁的附加信息
|-m|如果调用到本地方法的话，可以显示C/C++的堆栈

### JConsole - Java监视与管理控制台

在/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/bin/jconsole启动


### VisualVM - 多合一故障处理工具

在/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/bin/jvisualvm启动

> [VisualVM入门指南](https://visualvm.java.net/zh_CN/gettingstarted.html?Java_VisualVM)
