## 命令

### 普通命令-general

1. status

    > 查看hbase服务相关状态

    ```sql
    status
    1 active master, 0 backup masters, 1 servers, 5 dead, 8.0000 average load
    ```

2. table_help
    
    > 表相关命令的帮助文档

3. version
    
    > hbase服务版本号

4. whoami

    > 当前登录用户信息

### 数据定义语言-ddl

1. alter

    > 修改表，该表必须是禁用状态，否则要先禁用表

    ```sql
    ##修改或添加t1表中的f1列簇
    alter 't1', NAME => 'f1', VERSIONS => 5
    ##可以一次操作多个列簇
    alter 't1', 'f1', {NAME => 'f2', IN_MEMORY => true}, {NAME => 'f3', VERSIONS => 5}
    ##删除ns1空间中t1表的f1列簇
    alter 'ns1:t1', NAME => 'f1', METHOD => 'delete'
    alter 'ns1:t1', 'delete' => 'f1'
    ##也可以操作修改表中的MAX_FILESIZE, READONLY,MEMSTORE_FLUSHSIZE, DURABILITY属性等
    alter 't1', MAX_FILESIZE => '134217728'
    ##也可以删除表范围的属性
    alter 't1', METHOD => 'table_att_unset', NAME => 'MAX_FILESIZE'
    alter 't1', METHOD => 'table_att_unset', NAME => 'coprocessor$1'
    ```

2. alter_async

    > 异步修改，即改变列族的模式schema，不会等待所有的区域收到更改

    ```sql
    alter_async 't1', NAME => 'f1', VERSIONS => 5
    alter_async 'ns1:t1', NAME => 'f1', METHOD => 'delete'
    ```

3. alter_status

    > 修改并显示服务状态

4. create

    > 通过指定表名及一些列簇进行创建表

    ```sql
    create 't1', {NAME => 'f1'}, {NAME => 'f2'}, {NAME => 'f3'}
    #也可以简短表示
    create 't1', 'f1', 'f2', 'f3'
    #指定列簇及其选项
    create 't1', {NAME => 'f1', VERSIONS => 1, TTL => 2592000, BLOCKCACHE => true}
    #表的配置选项要放在最后面
    create 't1', 'f1', SPLITS => ['10', '20', '30', '40']
    create 't1', 'f1', SPLITS_FILE => 'splits.txt', OWNER => 'johndoe'
    create 't1', {NAME => 'f1', VERSIONS => 5}, METADATA => { 'mykey' => 'myvalue' }
    ```


5. describe

    > 查看表的结构

    ```sql
    describe 'test:user'
    #也可以简用
    desc 'test:user'
    ```

6. disable

    > 禁用表，`disable 'ns1:t1'`

7. disable_all

    > 禁用所有相匹配的表

    ```sql
    disable_all 't.*'
    disable_all 'ns:t.*'
    disable_all 'ns:.*'
    ```

8. drop
    
    > 删除表，该表首先被禁用

    ```sql
    drop 't1'
    drop 'ns1:t1'
    ```

9. drop_all

    > 删除与正则匹配的所有表

    ```sql
    drop_all 't.*'
    drop_all 'ns:t.*'
    drop_all 'ns:.*'
    ```

10. enable

    > 开始启用表，`enable 'ns1:t1'`

11. enable_all

    > 开始启用匹配的所有表，`enable_all 'ns1:t*'

12. exists

    > 指定的表是否存在，`exists 'ns1:t1'`

13. get_table

    > 获取指定的表，返回一个用户可以操作的表对象

    ```sql
    t1 = get_table 't1'
    t1 = get_table 'ns1:t1'
    t1.help
    ```

14. is_disabled

    > 指定的表是否不可用，`is_disabled 't1'`

15. is_enabled
    
    > 指定的表是否可用，`is_enabled 't1'`

16. list

    > 列出所有的表

17. locate_region

    > 定位给定的表及row key所在的区域

    ```sql
    locate_region 'test:event', '1'
    HOST                                         REGION
     30.28.186.7:61574                           {ENCODED => 33131d8d00af34459d0345e95091ece5, NAME => 'test:event,,1470801037110.33131d8d00af34459d0345e95091ece5.', STARTKEY =>'', ENDKEY => ''}
    ```

18. show_filters

    > 显示所有种类的过虑器

    ```sql
    show_filters
    DependentColumnFilter
    KeyOnlyFilter
    ColumnCountGetFilter
    SingleColumnValueFilter
    PrefixFilter
    SingleColumnValueExcludeFilter
    FirstKeyOnlyFilter
    ColumnRangeFilter
    TimestampsFilter
    FamilyFilter
    QualifierFilter
    ColumnPrefixFilter
    RowFilter
    MultipleColumnPrefixFilter
    InclusiveStopFilter
    PageFilter
    ValueFilter
    ColumnPaginationFilter
    ```

### 命名空间-namespace

1. alter_namespace

    > 修改命名空间属性

    ```sql
    #添加修改空间属性
    alter_namespace 'ns1', {METHOD => 'set', 'PROPERTY_NAME' => 'PROPERTY_VALUE'}
    #删除空间属性
    alter_namespace 'ns1', {METHOD => 'unset', NAME=>'PROPERTY_NAME'}
    ```

2. create_namespace

    > 创建命名空间及配置项

    ```sql
    create_namespace 'ns1', {'PROPERTY_NAME'=>'PROPERTY_VALUE'}
    ```

3. describe_namespace

    > 显示命名空间结构，`describe_namespace 'ns1'`

4. drop_namespace

    > 删除指定的命名空间,该空间必须是空的

5. list_namespace

    > 列表出所有的命名空间

6. list_namespace_tables
    
    > 列表指定命名空间下的所有表

### 数据操作语言-dml

1. append

    > 在指定的表行列位置追加值

    ```sql
    append 't1', 'r1', 'c1', 'value', ATTRIBUTES=>{'mykey'=>'myvalue'}
    append 't1', 'r1', 'c1', 'value', {VISIBILITY=>'PRIVATE|SECRET'}
    ```

2. count

    > 统计表的行数，`count 'ns1:t1'`

3. delete

    > 删除指定的表行列和时间的单元

    ```sql
    delete 't1', 'r1', 'c1', ts1
    delete 't1', 'r1', 'c1', ts1, {VISIBILITY=>'PRIVATE|SECRET'}
    ```

4. deleteall

    > 删除所指定行或列或时间的所有单元数据

    ```sql
    deleteall 'ns1:t1', 'r1'
    deleteall 't1', 'r1'
    deleteall 't1', 'r1', 'c1'
    deleteall 't1', 'r1', 'c1', ts1
    deleteall 't1', 'r1', 'c1', ts1, {VISIBILITY=>'PRIVATE|SECRET'}
    ```

5. get

    > 获取行或单元的内容

    ```sql
    get 'ns1:t1', 'r1'
    get 't1', 'r1'
    get 't1', 'r1', {TIMERANGE => [ts1, ts2]}
    get 't1', 'r1', {COLUMN => 'c1'}
    get 't1', 'r1', {COLUMN => ['c1', 'c2', 'c3']}
    get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1}
    get 't1', 'r1', {COLUMN => 'c1', TIMERANGE => [ts1, ts2], VERSIONS => 4}
    get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1, VERSIONS => 4}
    get 't1', 'r1', {FILTER => "ValueFilter(=, 'binary:abc')"}
    get 't1', 'r1', 'c1'
    get 't1', 'r1', 'c1', 'c2'
    get 't1', 'r1', ['c1', 'c2']
    get 't1', 'r1', {COLUMN => 'c1', ATTRIBUTES => {'mykey'=>'myvalue'}}
    get 't1', 'r1', {COLUMN => 'c1', AUTHORIZATIONS => ['PRIVATE','SECRET']}
    get 't1', 'r1', {CONSISTENCY => 'TIMELINE'}
    get 't1', 'r1', {CONSISTENCY => 'TIMELINE', REGION_REPLICA_ID => 1}
    ```

6. get_counter
7. get_splits

    > 获取指定表的分裂

8. incr

    > 以指定值自增指定表行列单元的值

    ```sql
    incr 'ns1:t1', 'r1', 'c1'
    incr 't1', 'r1', 'c1'
    incr 't1', 'r1', 'c1', 1
    ##每次加10增长
    incr 't1', 'r1', 'c1', 10
    incr 't1', 'r1', 'c1', 10, {ATTRIBUTES=>{'mykey'=>'myvalue'}}
    incr 't1', 'r1', 'c1', {ATTRIBUTES=>{'mykey'=>'myvalue'}}
    incr 't1', 'r1', 'c1', 10, {VISIBILITY=>'PRIVATE|SECRET'}
    ```

9. put

    > 给指定的表行列插入数据

    ```sql
    put 'ns1:t1', 'r1', 'c1', 'value'
    put 't1', 'r1', 'c1', 'value'
    put 't1', 'r1', 'c1', 'value', ts1
    put 't1', 'r1', 'c1', 'value', {ATTRIBUTES=>{'mykey'=>'myvalue'}}
    put 't1', 'r1', 'c1', 'value', ts1, {ATTRIBUTES=>{'mykey'=>'myvalue'}}
    put 't1', 'r1', 'c1', 'value', ts1, {VISIBILITY=>'PRIVATE|SECRET'}
    ```

10. scan

    > 扫描表，可以指定一个或多个条件：TIMERANGE, FILTER, LIMIT, STARTROW, STOPROW, ROWPREFIXFILTER, TIMESTAMP,MAXLENGTH or COLUMNS, CACHE or RAW, VERSIONS, ALL_METRICS or METRICS

    ```sql
      scan 'hbase:meta'
      scan 'hbase:meta', {COLUMNS => 'info:regioninfo'}
      scan 'ns1:t1', {COLUMNS => ['c1', 'c2'], LIMIT => 10, STARTROW => 'xyz'}
      scan 't1', {COLUMNS => ['c1', 'c2'], LIMIT => 10, STARTROW => 'xyz'}
      scan 't1', {COLUMNS => 'c1', TIMERANGE => [1303668804, 1303668904]}
      scan 't1', {REVERSED => true}
      scan 't1', {ALL_METRICS => true}
      scan 't1', {METRICS => ['RPC_RETRIES', 'ROWS_FILTERED']}
      scan 't1', {ROWPREFIXFILTER => 'row2', FILTER => "
         (QualifierFilter (>=, 'binary:xyz')) AND (TimestampsFilter ( 123, 456))"}
      scan 't1', {FILTER =>
             org.apache.hadoop.hbase.filter.ColumnPaginationFilter.new(1, 0)}
      scan 't1', {CONSISTENCY => 'TIMELINE'}
      #For setting the Operation Attributes
      scan 't1', { COLUMNS => ['c1', 'c2'], ATTRIBUTES => {'mykey' => 'myvalue'}}
      scan 't1', { COLUMNS => ['c1', 'c2'], AUTHORIZATIONS => ['PRIVATE','SECRET']}
    ```

11. truncate

    > 禁用删除重建指定的表

12. truncate_preserve

    > 禁用，删除并重新创建指定表的同时仍保持以前的区域边界。

### 工具-tools

1. assign
2. balance_switch
3. balancer
4. balancer_enabled
5. catalogjanitor_enabled,
6. catalogjanitor_run
7.  catalogjanitor_switch,
8. close_region,
9. compact,
10. compact_rs
11.  flush,
12. major_compact,
13. merge_region,
14. move,
15. normalize,
16. normalizer_enabled,
17. normalizer_switch,
18. split
19. trace
20.  unassign
21. wal_roll
22. zk_dump

### 复制同步-replication

1. add_peer,
2. append_peer_tableCFs
3. disable_peer
4. disable_table_replication
5. enable_peer
6. enable_table_replication
7. list_peers
8. list_replicated_tables
9. remove_peer
10. remove_peer_tableCFs
11. set_peer_tableCFs
12. show_peer_tableCFs

### 快照-snapshots

1. clone_snapshot
2. delete_all_snapshot
3. delete_snapshot
4. list_snapshots
5. restore_snapshot
6. snapshot

### 配置-configuration

1. update_all_config
2. update_config

### 引用-quotas

1. list_quotas
2. set_quota

### 安全-security

1. grant

    > 授予用户指定的权限

    `Syntax : grant <user>, <permissions> [, <@namespace> [, <table> [, <column family> [, <column qualifier>]]]`

    ```sql
    grant 'bobsmith', 'RWXCA'
    grant '@admins', 'RWXCA'
    grant 'bobsmith', 'RWXCA', '@ns1'
    grant 'bobsmith', 'RW', 't1', 'f1', 'col1'
    grant 'bobsmith', 'RW', 'ns1:t1', 'f1', 'col1'
    ```

2. list_security_capabilities
3. revoke, user_permission

    > 撤消用户的访问权限

    `Syntax : revoke <user> [, <@namespace> [, <table> [, <column family> [, <column qualifier>]]]]`

    ```sql
    revoke 'bobsmith'
    revoke '@admins'
    revoke 'bobsmith', '@ns1'
    revoke 'bobsmith', 't1', 'f1', 'col1'
    revoke 'bobsmith', 'ns1:t1', 'f1', 'col1'
    ```

4. user_permission

    > 显示特定用户的所有权限

    ```sql
    user_permission
    user_permission '@ns1'
    user_permission 'table1'
    user_permission 'namespace1:table1'
    user_permission '.*'
    user_permission '^[A-C].*'
    ```

### 过程-procedures

1. abort_procedure

    > 退出、舍弃存储过程

    ```sql
    abort_procedure proc_id
    abort_procedure proc_id, true
    abort_procedure proc_id, false
    ```

2.  list_procedures

    > 列出所有的存储过程

### 可视化标签-visibility labels

1. add_labels
2. clear_auths
3. get_auths
4. list_labels
5. set_auths
6. set_visibility
