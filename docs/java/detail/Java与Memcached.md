Java与Memcached通信的客户端比较常用的主要有com.googlecode.xmemcached和Memcached-Java-Client

## XMemcached

XMemcached是一个新java memcached client,是一个高性能的分布式内存对象的key-value缓存系统，用于动态Web应用以减轻数据库负载，现在也有很多人将它作为内存式数据库在使用，memcached通过它的自定义协议与客户端交互，而XMemcached就是它的一个java客户端实现。

### XMemcached主要特征

> 高性能

    XMemcached同样是基于java nio的客户端，java nio相比于传统阻塞io模型来说，有效率高（特别在高并发下）和资源耗费相对较少的优点。传统阻塞IO为了提高效率，需要创建一定数量的连接形成连接池，而nio仅需要一个连接即可（当然,nio也是可以做池化处理），相对来说减少了线程创建和切换的开销，这一点在高并发下特别明显。因此XMemcached与Spymemcached在性能都非常优秀，在某些方面（存储的数据比较小的情况下）Xmemcached比Spymemcached的表现更为优秀

> 支持完整的协议

Xmemcached支持所有的memcached协议，包括1.4.0正式开始使用的二进制协议

> 允许设置节点权重

XMemcached允许通过设置节点的权重来调节memcached的负载，设置的权重越高，该memcached节点存储的数据将越多，所承受的负载越大

> 支持客户端分布

Memcached的分布只能通过客户端来实现，XMemcached实现了此功能，并且提供了一致性哈希(consistent hash)算法的实现。

> 动态增删节点

XMemcached允许通过JMX或者代码编程实现节点的动态添加或者移除，方便用户扩展和替换节点等。

> 支持JMX

XMemcached通过JMX暴露的一些接口，支持client本身的监控和调整，允许动态设置调优参数、查看统计数据、动态增删节点等

> 与Spring框架的集成

鉴于很多项目已经使用Spring作为IOC容器，因此XMemcached也提供了对Spring框架的集成支持

> 客户端连接池

刚才已经提到java nio通常对一个memcached节点使用一个连接，而XMemcached同样提供了设置连接池的功能，对同一个memcached可以创建N个连接组成连接池来提高客户端在高并发环境下的表现，而这一切对使用者来说却是透明的。启用连接池的前提条件是保证数据之间的独立性或者数据更新的同步，对同一个节点的各个连接之间是没有做更新同步的，因此应用需要保证数据之间是相互独立的或者全部采用CAS更新来保证原子性。

> 可扩展性

XMemcached是基于java nio框架yanf4j实现的，因此在实现上结构相对清楚，分层比较明晰，在xmemcached 1.2.5之后已经将yanf4j合并到xmemcached，因此不再需要依赖yanf4j

### 应用

注意XMemcached依赖于slf4j，程序中的日志记录是用该接口来实现的，故要加上该依赖

#### 添加依赖

```xml
<!--slf4j会自动添加log4j的依赖-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.22</version>
</dependency>
<dependency>
    <groupId>com.googlecode.xmemcached</groupId>
    <artifactId>xmemcached</artifactId>
    <version>2.0.1</version>
</dependency>
```

#### spring配置

> 方式一:通过XMemcachedClientFactoryBean类__

```xml
<!--*********************************通过XMemcachedClientFactoryBean类，即可与spring框架集成**********************************-->
<!--需要权限认证的memcached服务器-->
<bean name="server1" class="java.net.InetSocketAddress">
    <constructor-arg>
        <value>127.0.0.1</value>
    </constructor-arg>
    <constructor-arg>
        <value>11212</value>
    </constructor-arg>
</bean>
<bean name="xmemcachedClient" class="net.rubyeye.xmemcached.utils.XMemcachedClientFactoryBean" destroy-method="shutdown">
    <!--memcached节点列表，形如“主节点1:port,备份节点1:port 主节点2:port,备份节点2:port“的字符串，可以不设置备份节点，主备节点逗号隔开，不同分组空格隔开。-->
    <property name="servers">
        <value>127.0.0.1:11211 127.0.0.1:11212</value>
    </property>
    <!--与servers对应的节点的权重-->
    <property name="weights">
        <list>
            <value>1</value>
            <value>2</value>
        </list>
    </property>
    <!--授权验证信息，仅在xmemcached 1.2.5及以上版本有效-->
    <property name="authInfoMap">
        <map> <entry key-ref="server1">
                <bean class="net.rubyeye.xmemcached.auth.AuthInfo" factory-method="typical">
                    <constructor-arg index="0">
                        <value>user</value>
                    </constructor-arg>
                    <constructor-arg index="1">
                        <value>password</value>
                    </constructor-arg>
                </bean>
            </entry>
        </map>
    </property>
    <!--nio连接池大小，默认为1-->
    <property name="connectionPoolSize" value="2"/>
    <!--协议工厂，net.rubyeye.xmemcached.command.BinaryCommandFactory,TextCommandFactory(默认),KestrelCommandFactory-->
    <property name="commandFactory">
        <bean class="net.rubyeye.xmemcached.command.BinaryCommandFactory"/>
    </property>
    <!--分布策略，一致性哈希net.rubyeye.xmemcached.impl.KetamaMemcachedSessionLocator或者ArraySessionLocator(默认)-->
    <property name="sessionLocator">
        <bean class="net.rubyeye.xmemcached.impl.KetamaMemcachedSessionLocator"/>
    </property>
    <!--序列化转换器，默认使用net.rubyeye.xmemcached.transcoders.SerializingTranscoder-->
    <property name="transcoder">
        <bean class="net.rubyeye.xmemcached.transcoders.SerializingTranscoder"/>
    </property>
    <!--IoBuffer分配器，默认为net.rubyeye.xmemcached.buffer.SimpleBufferAllocator，可选CachedBufferAllocator(不推荐)-->
    <property name="bufferAllocator">
        <bean class="net.rubyeye.xmemcached.buffer.SimpleBufferAllocator"/>
    </property>
    <!--是否启用failure模式，true为启用，默认不启用,所谓failure模式是指，当一个memcached节点down掉的时候，发往这个节点的请求将直接失败，而不是发送给下一个有效的memcached节点-->
    <property name="failureMode" value="false"/>
</bean>
<!--*********************************通过XMemcachedClientFactoryBean类，即可与spring框架集成**********************************-->

```

> 方式二:通过XmemcachedClientBuilder的工厂方法方式来创建MemcachedClient__

```xml
<!--*********************************通过XmemcachedClientBuilder的工厂方法方式来创建MemcachedClient，即可与spring框架集成**********************************-->
<!--定义工厂bean-->
<bean name="xMemcachedClientBuilder" class="net.rubyeye.xmemcached.XMemcachedClientBuilder">
    <!--服务器列表-->
    <constructor-arg>
        <list>
            <bean class="java.net.InetSocketAddress">
                <constructor-arg>
                    <value>127.0.0.1</value>
                </constructor-arg>
                <constructor-arg>
                    <value>11211</value>
                </constructor-arg>
            </bean>
            <bean class="java.net.InetSocketAddress">
                <constructor-arg>
                    <value>127.0.0.1</value>
                </constructor-arg>
                <constructor-arg>
                    <value>11212</value>
                </constructor-arg>
            </bean>
        </list>
    </constructor-arg>
    <!--对应服务器列表的权重列表-->
    <constructor-arg>
        <list>
            <value>1</value>
            <value>2</value>
        </list>
    </constructor-arg>
    <!--授权验证信息，仅在xmemcached 1.2.5及以上版本有效-->
    <property name="authInfoMap">
        <map>
            <entry key-ref="server1">
                <bean class="net.rubyeye.xmemcached.auth.AuthInfo" factory-method="typical">
                    <constructor-arg index="0">
                        <value>user</value>
                    </constructor-arg>
                    <constructor-arg index="1">
                        <value>password</value>
                    </constructor-arg>
                </bean>
            </entry>
        </map>
    </property>
    <!--nio连接池大小，默认为1-->
    <property name="connectionPoolSize" value="2"/>
    <!--协议工厂，net.rubyeye.xmemcached.command.BinaryCommandFactory,TextCommandFactory(默认),KestrelCommandFactory-->
    <property name="commandFactory">
        <bean class="net.rubyeye.xmemcached.command.TextCommandFactory"/>
    </property>
    <!--分布策略，一致性哈希net.rubyeye.xmemcached.impl.KetamaMemcachedSessionLocator或者ArraySessionLocator(默认)-->
    <property name="sessionLocator">
        <bean class="net.rubyeye.xmemcached.impl.KetamaMemcachedSessionLocator"/>
    </property>
    <!--序列化转换器，默认使用net.rubyeye.xmemcached.transcoders.SerializingTranscoder-->
    <property name="transcoder">
        <bean class="net.rubyeye.xmemcached.transcoders.SerializingTranscoder"/>
    </property>
</bean>
<!--通过工厂bean创建连接客户端-->
<bean name="xmemcachedClient1" factory-bean="xMemcachedClientBuilder" factory-method="build"
      destroy-method="shutdown"/>
<!--*********************************通过XmemcachedClientBuilder的工厂方法方式来创建MemcachedClient，即可与spring框架集成**********************************-->
```

#### 程序注入

```java
...
@Autowired
@Qualifier("xmemcachedClient")
private MemcachedClient client;
...

//设置值,它有三个参数，第一个是存储的key名称，第二个是expire时间（单位秒），超过这个时间,memcached将这个数据替换出去，0表示永久存储（默认是一个月），第三个参数就是实际存储的数据，可以是任意的java可序列化类型。
client.set('name',0,'bill');
//获取值时可以指定超时时间
client.get('name',3000);//3秒超时
//更新数据超时时间
client.touch(key,new-expire-time);
//获取缓存数据并更新超时时间
client.getAndTouch(key,new-expire-time);

//Memcached是通过cas协议实现原子更新，所谓原子更新就是compare and set，原理类似乐观锁，每次请求存储某个数据同时要附带一个cas值，memcached比对这个cas值与当前存储数据的cas值是否相等，如果相等就让新的数据覆盖老的数据，如果不相等就认为更新失败
GetsResponse<Integer> result = client.gets("a");
long cas = result.getCas(); 
//尝试将a的值更新为2
if (!client.cas("a", 0, 2, cas)) {
    System.err.println("cas error");
}

client.cas("a", 0, new CASOperation<Integer>() {
    public int getMaxTries() {
        return 1;
    }

    public Integer getNewValue(long currentCAS, Integer currentValue) {
        return 2;
    }
});

//迭代所有key
KeyIterator it=client.getKeyIterator(AddrUtil.getOneAddress("localhost:11211"));
while(it.hasNext())
{
   String key=it.next();
}

//Incr/Decr,incr和decr都有三个参数的方法，第一个参数指定递增的key名称，第二个参数指定递增的幅度大小，第三个参数指定当key不存在的情况下的初始值。两个参数的重载方法省略了第三个参数，默认指定为0。
assert(6==this.memcachedClient.incr("a", 5));
assert(10==this.memcachedClient.incr("a", 4));

//命名空间
//从1.4.2开始，xmemcached提供了memcached命名空间的封装使用，你可以将一组缓存项放到同一个命名空间下，可以让整个命名空间下所有的缓存项同时失效
//在全名空间下设置值
this.xmemcachedClient.withNamespace("namespace", new MemcachedClientCallable<Void>() {
    @Override
    public Void call(MemcachedClient memcachedClient) throws MemcachedException, InterruptedException, TimeoutException {
        memcachedClient.set("aa", 0, "aa");
        return null;
    }
});
//获取全名空间下的值
this.xmemcachedClient.withNamespace("namespace", new MemcachedClientCallable<String>() {
    @Override
    public String call(MemcachedClient memcachedClient) throws MemcachedException, InterruptedException, TimeoutException {
        return memcachedClient.get("aaa");
    }
});
//使得命名空间失效
this.xmemcachedClient.invalidateNamespace("namespace");


//查看统计信息
Map<InetSocketAddress,Map<String,String>> result=client.getStats();
//查看具体统计项信息
Map<InetSocketAddress,Map<String,String>> result=client.getStatsByItem("items");
```

详情可见：[XMemcached详细参考文档](https://github.com/killme2008/xmemcached/wiki/Xmemcached%20%E4%B8%AD%E6%96%87%E7%94%A8%E6%88%B7%E6%8C%87%E5%8D%97)

## Memcached-Java-Client

适合封装为一个工具类，对MemCachedClient进行封装一下使用

### 添加依赖

```xml
<!--memecached client-->
<dependency>
    <groupId>com.whalin</groupId>
    <artifactId>Memcached-Java-Client</artifactId>
    <version>3.0.2</version>
</dependency>
```

### 程序引用

```java
...
private static MemCachedClient client;
...
//初始化
static  {
    //创建连接池
    SockIOPool pool = SockIOPool.getInstance();
    //添加服务器
    pool.setServers(new String[]{"127.0.0.1:11211","127.0.0.1:11212"});
    //设置容错开关，设置为TRUE，当当前socket不可用时，程序会自动查找可用连接并返回，否则返回NULL，默认状态是true，建议保持默认
    pool.setFailover(true);
    //设置开始时每个cache服务器的可用连接数
    pool.setInitConn(10);
    //设置每个服务器最少可用连接数
    pool.setMinConn(5);
    //设置每个服务器最大可用连接数
    pool.setMaxConn(20);
    //设置连接池维护线程的睡眠时间,设置为0，维护线程不启动,维护线程主要通过log输出socket的运行状况，监测连接数目及空闲等待时间等参数以控制连接创建和关闭
    pool.setMaintSleep(30);
    //设置是否使用Nagle算法，因为我们的通讯数据量通常都比较大（相对TCP控制数据）而且要求响应及时，因此该值需要设置为false（默认是true）
    pool.setNagle(false);
    //设置socket的读取等待超时值
    pool.setSocketTO(3000);
    //设置socket的连接等待超时值
    pool.setAliveCheck(true);
    //设置连接失败恢复开关,设置为TRUE，当宕机的服务器启动或中断的网络连接后，这个socket连接还可继续使用，否则将不再使用，默认状态是true，建议保持默认
    pool.setFailback(true);
    //设置hash算法
    //alg=0 使用String.hashCode()获得hash code,该方法依赖JDK，可能和其他客户端不兼容，建议不使用
    //    alg=1 使用original 兼容hash算法，兼容其他客户端
    //    alg=2 使用CRC32兼容hash算法，兼容其他客户端，性能优于original算法
    //    alg=3 使用MD5 hash算法
    //    采用前三种hash算法的时候，查找cache服务器使用余数方法。采用最后一种hash算法查找cache服务时使用consistent方法
    pool.setHashingAlg(1);
    pool.initialize();
    client = new MemCachedClient();
    client.setDefaultEncoding("utf-8");
    client.setPrimitiveAsString(true);
    client.setCompressEnable(true);

}

@Override
public String get(String key) throws InterruptedException, MemcachedException, TimeoutException {

    return client.get(key).toString();
}

@Override
public void set(String key, String value) throws InterruptedException, MemcachedException, TimeoutException {
    client.set(key, value);
}
```
