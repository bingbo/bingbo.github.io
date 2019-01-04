非稳定参数是以-XX:开头的参数，使用-XX:+PrintFlagFinal参数可以输出所有参数的名称及默认值

#### 参数的使用方式有如下3种：

* -XX:+<option> 开启option参数
* -XX:-<option> 关闭option参数
* -XX:\<option>=\<value\> 将option参数的值设置为value

### 内存管理参数

|参数|默认值|使用介绍
|---|----|---
|DisableExplicitGC|默认关闭|忽略来自System.gc()方法触发的垃圾收集
|ExplicitGCInvokes|默认关闭|当收到System.gc()方法提交的垃圾收集申请时，使用CMS收集器进行收集
|UseSerialGC|Client模式的虚拟机默认开启，其他模式关闭|虚拟机运行在Client模式下的默认值，打开此开关后，使用Serial+Serial Old的收集器组合进行内存回收
|UseParNewGC|默认关闭|打开此开关后，使用ParNew+Serial Old的收集器组合进行内存回收
|UseConcMarkSweepGC|默认关闭|打开此开头后，使用ParNew+CMS+Serial Old的收集器组合进行内存回收，如果CMS收集器出现Concurrent Mode Failure,则Serial Old收集器将作为后备收集器
|UseParallelGC|Server模式的虚拟机默认开启，其他模式关闭|虚拟机运行在Server模式下的默认值，打开此开关后，使用Parallel Scavenge+Serial Old的收集器组合进行内存回收
|UseParallelOldGC|默认关闭|打开此开关后，使用Parallel Scavenge+Parallel Old的收集器组合进行内存回收
|SurvivorRatio|默认为8|新生代中Eden区域与Survivor区域的容量比值
|PretenureSizeThreshold|无默认值|直接晋升到老年代的对象大小，设置这个参数后，大于这个参数的对象将直接在老年代分配
|MaxTenuringThreshold|默认值为15|晋升到老年代的对象年龄，每个对象在坚持过一次Minor GC之后，年龄就增加1，当超过这个参数值里就进入老年代
|UseAdaptiveSizePolicy|默认开启|动态调整Java堆中各个区域的大小及进入老年代的年龄
|HandlePromotionFailure|JDK1.5及以前版本默认关闭，JDK1.6默认开启|是否允许分配担保失败，即老年代的剩余空间不足以应付新生代的整个Eden和Survivor区的所有对象存活的极端情况
|ParallelGCThreads|少于或等于8个CPU时默认值为CPU数量值，多于8个时比CPU数量值小|设置并行GC时进行内存回收的线程数
|GCTimeRatio|默认值为99|GC时间占总时间的比率，默认值为99，即允许1%的GC时间。仅在使用Parallel Scavenge收集器时生效
|MaxGCPauseMillis|无默认值|设置GC的最大 停顿时间，仅在使用Parallel Scavenge收集时生效
|CMSInitiatingOccupancyFraction|默认值为68|设置CMS收集器在老年代空间被使用多少后触发垃圾收集，仅在使用CMS收集器时生效
|UseCMSCompactAtFullCollection|默认开启|设置CMS收集器在完成垃圾收集后是否要进行一次内存碎片整理。仅在使用CMS收集器时生效
|CMSFullGCsBeforeCompaction|无默认值|设置CMS收集器在野德若干次垃圾收集后再启动一次内存碎片整理。仅在使用CMS收集器时生效
|ScavengeBeforeFullGC|默认开启|在FullGC发生之前 触发一次Minor GC
|UseGCOverheadLimit|默认开启|禁止GC过程无限制的执行，如果过于频繁，就直接发生OutOfMemory异常
|UseTLAB|Server模式默认开启|优先在本地线程缓冲区分配对象，避免分配内存时的锁定过程
|MaxHeapFreeRatio|默认值为70|当Xmx值比Xms值大时，堆可以动态收缩和扩展，这个参数控制当堆空闲大于指定比率时自动收缩
|MinHeapFreeRatio|默认值为40|当Xmx值比Xms值大时，堆可以动态收缩和扩展，这个参数控制当堆空闲小于指定比率时自动扩展
|MaxPermSize|大部分情况下默认值是64MB|永久代的最大值


### 即时编译参数

|参数|默认值|使用介绍
|---|----|---
|CompileThreshold|Client模式下默认值是1500，Server模式下是10000|触发方法即时编译的阈值
|OnStackReplacePercentage|Client模式下默认值是933，Server模式下是140|OSR比率，它是OSR即时编译阈值计算公式的一个参数，用于代替BackEdgeThreshold参数控制回边计数器的实际溢出阈值
|ReservedCodeCacheSize|大部分情况下默认值是32MB|即时编译器编译的代码缓存的最大值

### 类型加载参数

|参数|默认值|使用介绍
|---|----|---
|UseSplitVerifier|默认开启|使用依赖StackMapTable信息的类型检查 代替数据流分析，以加快字节码检验速度
|FailOverToOldVerifier|默认开启|当类型检验失败时，是否允许回到老的类型推导检验方式野德检验，如果开启则允许
|RelaxAccessControlCheck|默认关闭|在检验阶段放松对类型访问性的限制

### 多线程相关参数

|参数|默认值|使用介绍
|---|----|---
|UseSpinning|JDK1.6默认开启，JDK1.5默认关闭|开启自旋锁以避免线程频繁挂起和唤醒
|PreBlockSpin|默认值是10|使用自旋锁时默认的自旋次数
|UseThreadPriorities|默认开启|使用本地线程优先级
|UseBiasedLocking|默认开启|是否使用偏向锁，如果开启则使用
|UseFastAccessorMethods|默认开启|当频繁反射执行某个方法时，生成字节码来加快反射的执行速度

### 性能参数

|参数|默认值|使用介绍
|---|----|---
|AggressiveOpts|JDK1.6默认开启，JDK1.5默认关闭|使用激进的优化特性，这些特性一般是具备正面和负面双重影响的，需要根据具体应用特点分析才能判定是否对性能有好处
|UseLargePages|默认开启|如果可能，使用大内存分页，这项特性需要操作系统的支持
LargePageSizeInBytes|默认为4MB|使用指定大小的内存分页，这项特性需要操作系统 的支持
|StringCache|默认开启|是否使用字符串缓存，开启则使用


### 调试参数

|参数|默认值|使用介绍
|---|----|---
|HeapDumpOnOutOf MemoryError|默认关闭|在发生内存溢出异常时是否生成堆转储快照，关闭则不生成
|OnOutOfMemoryError|无默认值|当虚拟机抛出内存溢出异常时，执行指定的命令
|OnError|无默认值|当虚拟机抛出Error异常时，执行指定的命令
|PrintClassHistogram|默认关闭|使用ctrl-break快捷键输出类统计状态，相当于jmap-histo的功能
|PrintConcurrentLocks|默认关闭|打印J.U.C中锁的状态
|PrintCommandLineFlags|默认关闭|打印启动虚拟机时输入的非稳定参数
|PrintCompilation|默认关闭|打印方法即时编译信息
|PrintGC|默认关闭|打印GC信息
|PrintGCDetails|默认关闭|打印GC的详细信息
|PrintGCTimeStamps|默认关闭|打印GC停顿耗时
|PrintTenuringDistribution|默认关闭|打印GC后新生代各个年龄对象的大小
|TraceClassLoading|默认关闭|打印类加载信息
|TraceClassUnloading|默认关闭|打印类卸载信息
|PrintInlining|默认关闭|打印方法的内联信息
|PrintCFGToFile|默认关闭|将CFG图信息输出到文件，只有DEBUG版虚拟机才支持此参数
|PrintIdealGraphFile|默认关闭|将Ideal图信息输出到文件，只有DEBUG版虚拟机才支持此参数
|UnlockDiagnosticVM Options|默认关闭|让虚拟机进入诊断模式，一些参数需要在诊断模式中才能使用
|PrintAssembly|默认关闭|打印即时编译后的二进制信息
