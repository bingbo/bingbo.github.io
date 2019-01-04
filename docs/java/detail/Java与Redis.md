### 配置依赖

```xml
<!--redis client-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <type>jar</type>
    <scope>compile</scope>
</dependency>
```

### 通过程序封装应用

```java
public class Redis {


    //简单应用
    public static Jedis getInstance() {
        Jedis redis = new Jedis("127.0.0.1");
        return redis;
    }

    public static Jedis getInstance(String host, int port) {
        return new Jedis(host, port);
    }

    public static Jedis getInstance(String host, int port, String password) {
        Jedis redis = new Jedis(host, port);
        if (!StringUtils.isEmpty(password)) {
            redis.auth(password);
        }
        return redis;
    }

    //通过根据配置建立Jedis连接池，获取连接
    public static Jedis getInstance(JedisPoolConfig config, String host, int port) {
        JedisPool pool = null;
        if (null == config) {
            //建立连接池配置参数
            config = new JedisPoolConfig();
            //设置最大连接数
            config.setMaxTotal(100);
            //设置最大空闲连接数
            config.setMaxIdle(10);
            //设置最大阻塞时间，单位许毫秒
            config.setMaxWaitMillis(1000);
        }
        //同步创建连接池
        synchronized (pool) {
            if (null == pool) {
                pool = new JedisPool(config, host, port);
            }
        }
        return pool.getResource();
    }

    //Jedis 实现分片,Jedis分片采用Hash算法和基于的Key模式匹配。Jedis定义一个Hash接口，如果觉得自带的不爽，可以自己实现一个Hash算法。Jedis自带的Hash的算法是MurmurHash 2.0 。
    public static ShardedJedis getInstance(List<JedisShardInfo> shards) {
        if (null == shards) {
            //建立服务器列表
            shards = new ArrayList<JedisShardInfo>();

            //添加第一台服务器信息
            JedisShardInfo si = new JedisShardInfo("localhost", 6379);
            si.setPassword("123");
            shards.add(si);

            //添加第二台服务器信息
            si = new JedisShardInfo("localhost", 6399);
            si.setPassword("123");
            shards.add(si);
        }
        //建立分片连接对象
        ShardedJedis jedis = new ShardedJedis(shards);

        //建立分片连接对象,并指定Hash算法
        //ShardedJedis jedis = new ShardedJedis(shards,selfHash);
        return jedis;
    }

    //分片也可以支持连接池
    public static ShardedJedis getInstance(JedisPoolConfig config, List<JedisShardInfo> shards) {
        ShardedJedisPool pool = new ShardedJedisPool(config, shards);
        return pool.getResource();
    }

}
```

### Spring配置

```xml
<!--*********************redis config*************************-->
<!--配置分片连接池-->
<bean name="jedisPool" class="redis.clients.jedis.ShardedJedisPool" destroy-method="close">
    <constructor-arg name="poolConfig">
        <bean class="redis.clients.jedis.JedisPoolConfig">
            <property name="maxTotal" value="10"/>
            <property name="maxWaitMillis" value="1000"/>
            <property name="maxIdle" value="10"/>
            <property name="blockWhenExhausted" value="true"/>
            <property name="testOnBorrow" value="true"/>
        </bean>
    </constructor-arg>
    <constructor-arg name="shards">
        <list>
            <bean class="redis.clients.jedis.JedisShardInfo">
                <constructor-arg name="host">
                    <value>localhost</value>
                </constructor-arg>
                <constructor-arg name="port">
                    <value>6379</value>
                </constructor-arg>
                <!--<property name="password" value=""/>-->
            </bean>
            <bean class="redis.clients.jedis.JedisShardInfo">
                <constructor-arg name="host">
                    <value>localhost</value>
                </constructor-arg>
                <constructor-arg name="port">
                    <value>6399</value>
                </constructor-arg>
                <!--<property name="password" value=""/>-->
            </bean>
        </list>
    </constructor-arg>
</bean>
<!--*********************redis config*************************-->
```

### 程序注入调用

```java
@Service
public class RedisServiceImpl implements RedisService {

    @Autowired
    @Qualifier("jedisPool")
    private ShardedJedisPool pool;


    @Override
    public boolean set(String key, String value) {
        String result = pool.getResource().set(key, value);
        return true;
    }

    @Override
    public Object get(String key) {
        return pool.getResource().get(key);
    }
}
```

详情见[Jedis文档](https://github.com/xetorthio/jedis)