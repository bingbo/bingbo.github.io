## memcached

* 性能高，不是瓶颈，memcached可以利用多核优势，单实例吞吐量极高，可达几十万QPS，适用于最大程度扛量
* 只支持简单的key/value数据结构
* 无法进行持久化，数据不能备份，只能用于缓存使用，且重启后数据全部丢失
* 数据无法同步迁移
* 内存分配采用Slab Allocation机制管理内存，value大小分布差异较大时会造成内存利用率降低，并引发低利用率时依然出现踢出等问题。需要用户注重value设计
* 在并发场景下，用cas保证一致性


## redis

* 支持多种数据结构，如string(字符串),list(双向链表),dict(hash表),set(集合),zset(排序set),hyperloglogs(基数估算)，在string类型上会消耗较多内存，可以使用dict(hash表)压缩存储惟降低内存耗用
* 多数据结构可减少网络IO次数
* 支付持久化操作，可以进行aof及rdb数据持久化到磁盘，进行数据备份或数据恢复等
* 支持replication数据复制，通过master-slave机制，可以实时进行数据的同步复制，支持多级复制和增量复制
* 单线程，所有命令串行执行，并发情况下不需要考虑数据一致性问题，性能受限于CPU性能，单实例最高可达5-6万QPS
* 支持消息订阅机制，可以用来进行消息订阅与通知
* 支持简单的事务，只保证事务中的每个操作连续执行

## mongodb

* 文档形数据库，可以存放xml、json、bson类型数据
* 支持丰富的数据表达，索引，查询语言丰富
* QPS比redis、memcached相对小
* 适合大数据量存储，依赖操作系统vm做内存管理，吃内存比较厉害，最好不要与别的服务在一起
* 支持master-slave,replication机制，采用binlog方式进行持久化
* 不支持事务，内置了数据分析功能


_应用场景_

> redis和memcached在性能上都比较高，redis在存储小数据时比memcached性能更高，但在100k以上的数据中，memcached性能要高于redis；   

> 在内存使用效率上，如果kv存储memcached内存利用率更高，而如果redis采用hash结构来做kv存储，由于其组合式压缩，内存利用率高于memcached；   

> 在利用多种数据结构操作数据时，memcached需要将数据在客户端操作，增加了网络IO开销，故性能低于redis;在存储大量数据时，mongodb更适合存储海量数据
