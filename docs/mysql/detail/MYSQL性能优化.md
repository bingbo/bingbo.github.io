## 性能优化工具

### explain

```sql
mysql> explain select * from cart where (user_id = 11111 or session_id = '') and goods_id = 111 and is_deleted = 0 order by create_time;        
+----+-------------+-------------+------+------------------------------------------------------------------------+---------------------------+---------+-------+------+-------------+
| id | select_type | table       | type | possible_keys                                                          | key                       | key_len | ref   | rows | Extra       |
+----+-------------+-------------+------+------------------------------------------------------------------------+---------------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | cart | ref  | ind_isdeleted_create_time,ind_session_id_goods_id,ind_user_id_goods_id | ind_isdeleted_create_time | 1       | const |    5 | Using where |
+----+-------------+-------------+------+------------------------------------------------------------------------+---------------------------+---------+-------+------+-------------+
1 row in set (0.00 sec)
```

### show profiles

* 打开profiles，`SET profiling = 1;`
* 查看所有已执行语句的情况

    ```sql
    mysql> show profiles\G
    *************************** 1. row ***************************
    Query_ID: 1
    Duration: 0.00045025
    Query: select * from cart where (user_id = 11111 or session_id = '') and goods_id = 111 and is_deleted = 0 order by create_time
    1 row in set (0.00 sec)
    ```
    
* 查看某条已执行语句的情况，`show profile for query 2;`
* 查看某条已执行语句的cpu、内存、io等情况

    ```sql
    show profile cpu,block io,memory,swaps,context switches,source for query 1;
    ```
    
## 注意点

* order by xxx 要用到索引，即xxx建立的索引，则xxx必须在select中出现，同时xxx字段必须不为空，即not null，如explain select create_time from user order by create_time desc;
* 推荐使用InnoDB表，InnoDB在并发读写方面好于MyISAM、支持事务、表不易损坏
* 不推荐MySQL本身不擅长或有性能问题的功能，包括: 触发器(Triger) 、存储过程(Procedure)、外键(Foreign Key)、函数(Function)、视图(Views)、事件(Event)等，主要原因在于与业务耦合太大，或是某些场景下可能造成主从不一致，或是不利于线上追查问题，或是不利于统一运维等
* 不要建立过多索引，索引越多，插入性能越低，磁盘占用空间越多，推荐索引个数不要超过7个 不建议索引个数超过字段个数
* 建表方面：

    * 要有COMMENT，建立表及字段的COMMENT是一个天然的功能说明书
    * 字段设置为非NULL，更有利于语句查询，规避一些容易出现的问题,InnoDB本身对NULL的处理有别于其它正常数据或空数据
    * 禁用ENUM,SET：原因在于不利于扩展，扩展变更表会导致表阻塞写
    * VARCHAR长度的选择,以UTF8不超过2600字符,GBK不超过4000字符为最佳，在业务中推荐不过7000字符长度
    * 不推荐使用BLOB,TEXT等大字段：主要原因在于大字段带来更多的IO,网络开销
    * 时间字段在大表中(百万级)推荐使用UNSIGNED INT以减少存储空间
    * IP字段在大表中(百万级) 推荐使用UNSIGNED INT以减少存储空间
    * 分区表上限数据1亿行，表分区数上限10个，磁盘占用上限50G,相关查询必须有分区键。主要出于对分区键、文件句柄、SQL性能等的开销考虑，分区表上运行的SQL必须包含分区键

* 禁止使用SELECT * 获取所有字段，会导致更多的网络及可能更多的IO消耗
* INSERT需要指明需要插入的字段名。表字段变更将直接导致原有正常运行的语句变成错误语句。也易出现因拼写不严禁导致的数据插错字段
* 数字与字符之间不做等值查询，如字段是INT类型，结果查询使用ID=’1’这种情况。导致无法利用到索引
* INSERT ON DUPLICATE KEY语句在5.0及以下版本中是串行执行的，其依赖于表中唯一索引，确保唯一索引不变更
* JOIN表不推荐超过2个 并不是说JOIN3个或以上就一定会出问题，主要原因在于JOIN表越多，性能消耗越大,系统并发能力越低