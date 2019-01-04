### conf/hbase-env.sh

配置系统的部署信息和环境变量。 -- 这个配置会被启动shell使用 -- 然后在XML文件里配置信息，覆盖默认的配置。告知HBase使用什么目录地址，ZooKeeper的位置等等信息

### conf/log4j.properties

日志相关信息配置

### conf/hbase-site.xml

hbase的配置都在这个文件里，文档是用hbase默认配置文件生成的，文件源是 hbase-default.xml,不是所有的配置都在 hbase-default.xml出现.只要改了代码，配置就有可能改变，所以唯一了解这些被改过的配置的办法是读源代码本身。

要注意的是，要重启集群才能是配置生效。

```xml
<configuration>
    <!--数据存放目录-->
    <property> 
        <name>hbase.rootdir</name>
        <value>file:///Users/zhangbingbing/Work/hbase-1.2.1/data</value> 
    </property>
    <!-- 写缓存大小，越大占内存就大，但可以减少RPC次数-->
    <property>
        <name>hbase.client.write.buffer</name>
        <value>2097152</value>
    </property>
    <!--当调用Scanner的next方法，而值又不在缓存里的时候，从服务端一次获取的行数。越大的值意味着Scanner会快一些，但是会占用更多的内存。当缓冲被占满的时候，next方法调用会越来越慢。慢到一定程度-->
    <property>
        <name>hbase.client.scanner.caching</name>
        <value>1</value>
    </property>
    <!--一个KeyValue实例的最大size.这个是用来设置存储文件中的单个entry的大小上界。因为一个KeyValue是不能分割的，所以可以避免因为数据过大导致region不可分割。明智的做法是把它设为可以被最大region size整除的数。如果设置为0或者更小，就会禁用这个检查。默认10MB-->
    <property>
        <name>hbase.client.keyvalue.maxsize</name>
        <value>10485760</value>
    </property>
    <!-- regionserver与zookeeper的连接超时时间，当超时时间到后，ReigonServer会被Zookeeper从RS集群清单中移除，HMaster收到移除通知后，会对这台server负责的regions重新balance，让其他存活的RegionServer接管.-->
    <property>
        <name>zookeeper.session.timeout</name>
        <value>60000</value>
    </property>
    <!--RegionServer的请求处理IO线程数,这个参数的调优与内存息息相关。 较少的IO线程，适用于处理单次请求内存消耗较高的Big PUT场景（大容量单次PUT或设置了较大cache的scan，均属于Big PUT）或ReigonServer的内存比较紧张的场景。 较多的IO线程，适用于单次请求内存消耗低，TPS要求非常高的场景-->
    <property>
        <name>hbase.regionserver.handler.count</name>
        <value>10</value>
    </property>
    <!--在当前ReigonServer上单个Reigon的最大存储空间，单个Region超过该值时，这个Region会被自动split成更小的region,小region对split和compaction友好，因为拆分region或compact小region里的storefile速度很快，内存占用低。缺点是split和compaction会很频繁，一般512以下的都算小region。-->
    <property>
        <name>hbase.hregion.max.filesize</name>
        <value>256</value>
    </property>
    <!--减少因内存碎片导致的Full GC，提高整体性能-->
    <property>
        <name>hbase.hregion.memstore.mslab.enabled</name>
        <value>true</value>
    </property>
    <!--storefile的读缓存占用Heap的大小百分比，0.2表示20%。该值直接影响数据读的性能调优：当然是越大越好，如果写比读少很多，开到0.4-0.5也没问题-->
    <property>
        <name>hfile.block.cache.size</name>
        <value>0.40</value>
    </property>
    <!--2:说明：当一个region里的memstore占用内存大小超过hbase.hregion.memstore.flush.size两倍的大小时，block该region的所有请求，进行flush，释放内存。调优： 这个参数的默认值还是比较靠谱的。如果你预估你的正常应用场景（不包括异常）不会出现突发写或写的量可控，那么保持默认值即可-->
    <property>
       <name>hbase.hregion.memstore.block.multiplier</name>name>
       <value>2</value>value>
   </property>
   <!--默认: 67108864 :当memstore的大小超过这个值的时候，会flush到磁盘-->
   <property>
       <name>hbase.hregion.memstore.flush.size</name>
       <value>134217728</value>
   </property>

    <!--Zookeeper集群的地址列表，用逗号分割-->
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>{zk-address}</value>
    </property>
    <!--ZooKeeper中的HBase的根ZNode。所有的HBase的ZooKeeper会用这个目录配置相对路径。默认情况下，所有的HBase的ZooKeeper文件路径是用相对路径，所以他们会都去这个目录下面。默认: /hbase-->
    <property>
        <name>zookeeper.znode.parent</name>
        <value>/hbase</value>
    </property>
    <!--ZooKeeper的zoo.conf中的配置。 客户端连接的端口-->
    <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
    </property>

    <!-- 失败时的重试次数-->
    <property>
        <name>hbase.client.retries.number</name>
        <value>5</value>
    </property>
    <!--通常的客户端暂停时间。最多的用法是客户端在重试前的等待时间。比如失败的get操作和region查询操作等都很可能用到。默认: 1000-->
    <property>
        <name>hbase.client.pause</name>
        <value>1000</value>
    </property>
    <!--线程池的最大线程数，当等待的请求阻塞时，就创建新的线程处理请求，直到达到最大值，否则丢掉相应的连接-->
    <property>
        <name>hbase.thrift.maxWorkerThreads</name>
        <value>200</value>
    </property>

    <!--线程池的最小线程数，当等待的请求阻塞时，就创建新的线程处理请求，直到达到最大值，否则丢掉相应的连接-->
    <property>
        <name>hbase.thrift.minWorkerThreads</name>
        <value>1000</value>
    </property>
    <!--线程保持连接的最大 时间-->
    <property>
        <name>hbase.thrift.threadKeepAliveTimeSec</name>
        <value>60</value>
    </property>

    <property>
        <name>hbase.thrift.port</name>
        <value>9090</value>
    </property>
</configuration>
```
