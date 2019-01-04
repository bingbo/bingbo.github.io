MySQL 5.7提供了一系列的方法去修改分区表，有添加、删除、重定义、合并或拆分分区等，所有这些行为都可以用`ALTER TABLE`语句来实现，也有多种方法获取有关分区表或分区的信息。

修改表的分区方案，必须用带partition_options语法的`alter table`语句，一般用`PARTITION BY`。如分区表trb3：

```sql
CREATE TABLE trb3 (id INT, name VARCHAR(50), purchased DATE)
    PARTITION BY RANGE( YEAR(purchased) ) (
        PARTITION p0 VALUES LESS THAN (1990),
        PARTITION p1 VALUES LESS THAN (1995),
        PARTITION p2 VALUES LESS THAN (2000),
        PARTITION p3 VALUES LESS THAN (2005)
    );
```

修改为：```sql ALTER TABLE trb3 PARTITION BY KEY(id) PARTITIONS 2;```

`ALTER TABLE ... ENGINE = ... `仅仅会修改表的存储引擎而保持分区方案完好无损。`ALTER TABLE ... REMOVE PARTITIONING`则删除表的分区。

> 仅`PARTITION BY, ADD PARTITION, DROP PARTITION, REORGANIZE PARTITION, or COALESCE PARTITION`短语用于`ALTER TABLE`语句，如果你要删除一个分区并重新组织剩下的分区，则你必须执行两步单独的`ALTER TABLE`语句，一个带`drop partition`，另一个带`reorganize partitions`。

使用`ALTER TABLE ... TRUNCATE PARTITION`语句删除选定的分区的所有行。

## LIST和RANGE分区管理

删除p2分区：

```sql
mysql> ALTER TABLE tr DROP PARTITION p2;
Query OK, 0 rows affected (0.03 sec)
```

如果你删除了一个分区，是你也删除了该分区里的所有数据。所以你必须有删除权限。

如果您希望从所有分区删除所有数据，同时保留表的定义和划分方案，使用TRUNCATE TABLE语句

如果你想要修改表的分区而不丢失数据，则使用`alter table ... reorganize partition`代替。

删除LIST分区和删除RANGE分区一样使用`alter table ... drop partition`语法，但有一点不同的是：你不能再向删除的分区中插入相应的数据了。

给分区表添加新的分区，则使用`alter table ... add partition`语句，针对RANGE分区表，你可以添加新分区在已有分区后面。

```sql
CREATE TABLE members (
    id INT,
    fname VARCHAR(25),
    lname VARCHAR(25),
    dob DATE
)
PARTITION BY RANGE( YEAR(dob) ) (
    PARTITION p0 VALUES LESS THAN (1970),
    PARTITION p1 VALUES LESS THAN (1980),
    PARTITION p2 VALUES LESS THAN (1990)
);

ALTER TABLE members ADD PARTITION (PARTITION p3 VALUES LESS THAN (2000));
```

尝试在已有分区之前或之间添加新分区，则会出错：

```sql
mysql> ALTER TABLE members
    > ADD PARTITION (
    > PARTITION n VALUES LESS THAN (1960));
ERROR 1463 (HY000): VALUES LESS THAN value must be strictly »
increasing for each partition
```

不过你可以重新组织第一个分区为两个新的分区，如：

```sql
ALTER TABLE members
    REORGANIZE PARTITION p0 INTO (
        PARTITION n0 VALUES LESS THAN (1960),
        PARTITION n1 VALUES LESS THAN (1970)
);
```

你也可以添加新分区到LIST分区表上：

```sql
CREATE TABLE tt (
    id INT,
    data INT
)
PARTITION BY LIST(data) (
    PARTITION p0 VALUES IN (5, 10, 15),
    PARTITION p1 VALUES IN (6, 12, 18)
);

ALTER TABLE tt ADD PARTITION (PARTITION p2 VALUES IN (7, 14, 21));
```

注意这里新添加的分区里不能包含已在其他分区里的列值。如：

```sql
mysql> ALTER TABLE tt ADD PARTITION
> (PARTITION np VALUES IN (4, 8, 12));
ERROR 1465 (HY000): Multiple definition of same constant »
in list partitioning
```

你也可以添加多个分区在一个`ALTER TABLE ... ADD PARTITION`语句中：

```sql
ALTER TABLE employees ADD PARTITION (
    PARTITION p5 VALUES LESS THAN (2010),
    PARTITION p6 VALUES LESS THAN MAXVALUE
);
```

幸运的是，你可以重新定义分区方案也不丢失数据，如下所示的RANGE分区表：

```sql
mysql> SHOW CREATE TABLE members\G
*************************** 1. row ***************************
Table: members
Create Table: CREATE TABLE `members` (
    `id` int(11) default NULL,
    `fname` varchar(25) default NULL,
    `lname` varchar(25) default NULL,
    `dob` date default NULL
) ENGINE=MyISAM DEFAULT CHARSET=latin1
PARTITION BY RANGE ( YEAR(dob) ) (
    PARTITION p0 VALUES LESS THAN (1970) ENGINE = MyISAM,
    PARTITION p1 VALUES LESS THAN (1980) ENGINE = MyISAM,
    PARTITION p2 VALUES LESS THAN (1990) ENGINE = MyISAM.
    PARTITION p3 VALUES LESS THAN (2000) ENGINE = MyISAM
)
```

拆分分区：

```sql
ALTER TABLE members REORGANIZE PARTITION p0 INTO (
    PARTITION s0 VALUES LESS THAN (1960),
    PARTITION s1 VALUES LESS THAN (1970)
);
```

合并分区：

```sql
ALTER TABLE members REORGANIZE PARTITION s0,s1 INTO (
    PARTITION p0 VALUES LESS THAN (1970)
);
```

通过`REORGANIZE PARTITION`语句进行拆分或合并分区并不会丢失数据，其语法如下所示：

```sql
ALTER TABLE tbl_name
    REORGANIZE PARTITION partition_list
    INTO (partition_definitions);
```

同样也可用于LIST分区表：

```sql
ALTER TABLE tt ADD PARTITION (PARTITION np VALUES IN (4, 8));
ALTER TABLE tt REORGANIZE PARTITION p1,np INTO (
    PARTITION p1 VALUES IN (6, 18),
    PARTITION np VALUES in (4, 8, 12)
);
```

这里有几点需要注意，当使用`ALTER TABLE ... REORGANIZE PARTITION`语句来重新分区RANGE和LIST分区表时：

* 这些`PARTITION`短语用于决定新分区方案时与在`create table`语句中使用是一样的，重要的是要记住新的分区方案不能有重叠的区域或集合
* 分区组合列表应该与之前的区域或集合保持一致
* 对于通过RANGE分区的表，你可以重新组织只有相邻分区;你不能跳过几个范围进行分区
* 你不能使用`reorganize partition`去改变表的分区类型，不过可以使用`ALTER TABLE ... PARTITION BY ....`来改：
    
    ```sql
    ALTER TABLE members
        PARTITION BY HASH( YEAR(dob) )
        PARTITIONS 8;
    ```

## HASH和KEY分区管理

你不能像删除RANGE和LIST分区表一样删除HASH或KEY分区表中的分区。然而你可以合并HASH和KEY分区使用`alter table ... coalesce partition`语句。

```sql
CREATE TABLE clients (
    id INT,
    fname VARCHAR(30),
    lname VARCHAR(30),
    signed DATE
)
PARTITION BY HASH( MONTH(signed) )
PARTITIONS 12;

mysql> ALTER TABLE clients COALESCE PARTITION 4;
Query OK, 0 rows affected (0.02 sec)
    ```

COALESCE一样也使用于HASH, KEY, LINEAR HASH, or LINEAR KEY分区表：

```sql
mysql> CREATE TABLE clients_lk (
    -> id INT,
    -> fname VARCHAR(30),
    -> lname VARCHAR(30),
    -> signed DATE
    -> )
-> PARTITION BY LINEAR KEY(signed)
-> PARTITIONS 12;
Query OK, 0 rows affected (0.03 sec)
mysql> ALTER TABLE clients_lk COALESCE PARTITION 4;
Query OK, 0 rows affected (0.06 sec)
Records: 0 Duplicates: 0 Warnings: 0
```

如果你尝试删除比现有分区多的分区，则会报错：

```sql
mysql> ALTER TABLE clients COALESCE PARTITION 18;
ERROR 1478 (HY000): Cannot remove all partitions, use DROP TABLE instead
```

为了增加分区从12到18,则可以使用`alter table ... add partition`，如下所示：

```sql
ALTER TABLE clients ADD PARTITION PARTITIONS 6;
```

## 交换表的分区与子分区

交换一个表的分区或子分区与另一个表，使用`alter table pt exchange partition p with table nt`，该语句有以下特点：

1. 表nt是没有分区的
1. 表nt不是临时表
1. 表pt和nt的结构相同
1. 表nt没有外键，别的表也没有外键到nt表
1. 表nt中没有行是在表p定义的分区之外的。

使用该语句时要注意以下几点：

* 执行该语句不会触发触发器不管是分区表还是要被交换的表
* 在交换表中的任何auto_increment列会被重转置
* IGNORE关键字与该语句使用是没有任何效果

```sql
ALTER TABLE pt
EXCHANGE PARTITION p
WITH TABLE nt;
```

你可以追加`with validation`或`without validation`等短语，当`without validation`指定时，该语句不执行一行一行的验证当与未分区的表交换分区时，`with validation`是默认选项，不需要明确指定。

### 与未分区表交换分区

假如有表：

```sql
CREATE TABLE e (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30)
)
PARTITION BY RANGE (id) (
    PARTITION p0 VALUES LESS THAN (50),
    PARTITION p1 VALUES LESS THAN (100),
    PARTITION p2 VALUES LESS THAN (150),
    PARTITION p3 VALUES LESS THAN (MAXVALUE)
);
INSERT INTO e VALUES
    (1669, "Jim", "Smith"),
    (337, "Mary", "Jones"),
    (16, "Frank", "White"),
    (2005, "Linda", "Black");
```

创建类似的未分区表e2：

```sql
mysql> CREATE TABLE e2 LIKE e;
Query OK, 0 rows affected (1.34 sec)
mysql> ALTER TABLE e2 REMOVE PARTITIONING;
Query OK, 0 rows affected (0.90 sec)
Records: 0 Duplicates: 0 Warnings: 0
```

此时表e的分区情况如下，p0分区已有一条记录：

```sql
mysql> SELECT PARTITION_NAME, TABLE_ROWS
-> FROM INFORMATION_SCHEMA.PARTITIONS
-> WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0 | 1 |
| p1 | 0 |
| p2 | 0 |
| p3 | 3 |
+----------------+------------+
4 rows in set (0.00 sec)
```

与表e2交换表e的p0分区：

```sql
mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2;
Query OK, 0 rows affected (0.28 sec)
```

结果如下所示，表e的p0分区的一条记录已在e2表里：

```sql
mysql> SELECT PARTITION_NAME, TABLE_ROWS
    -> FROM INFORMATION_SCHEMA.PARTITIONS
    -> WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0 | 0 |
| p1 | 0 |
| p2 | 0 |
| p3 | 3 |
+----------------+------------+
4 rows in set (0.00 sec)


mysql> SELECT * FROM e2;
+----+-------+-------+
| id | fname | lname |
+----+-------+-------+
| 16 | Frank | White |
+----+-------+-------+
1 row in set (0.00 sec)
```

交换后的分区不一定会是空的，如先插入一条到e中：

```sql
mysql> INSERT INTO e VALUES (41, "Michael", "Green");
Query OK, 1 row affected (0.05 sec)
mysql> SELECT PARTITION_NAME, TABLE_ROWS
    -> FROM INFORMATION_SCHEMA.PARTITIONS
    -> WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0 | 1 |
| p1 | 0 |
| p2 | 0 |
| p3 | 3 |
+----------------+------------+
4 rows in set (0.00 sec)
```

再次交换p0分区：

```sql
mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2;
Query OK, 0 rows affected (0.28 sec)
```

结果如下：

```sql
mysql> SELECT * FROM e;
+------+-------+-------+
| id | fname | lname |
+------+-------+-------+
| 16 | Frank | White |
| 1669 | Jim | Smith |
| 337 | Mary | Jones |
| 2005 | Linda | Black |
+------+-------+-------+
4 rows in set (0.00 sec)


mysql> SELECT PARTITION_NAME, TABLE_ROWS
    -> FROM INFORMATION_SCHEMA.PARTITIONS
    -> WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0 | 1 |
| p1 | 0 |
| p2 | 0 |
| p3 | 3 |
+----------------+------------+
4 rows in set (0.00 sec)
mysql> SELECT * FROM e2;
+----+---------+-------+
| id | fname | lname |
+----+---------+-------+
| 41 | Michael | Green |
+----+---------+-------+
1 row in set (0.00 sec)
```

### 不匹配的行

任何未分区表中的行必须满足`alter table ... exchange partition`语句指定分区的条件，否则该语句会失败,如有行ID已大于p0分区的最大值49,所以会交换失败：

```sql
mysql> INSERT INTO e2 VALUES (51, "Ellen", "McDonald");
Query OK, 1 row affected (0.08 sec)
mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2;
ERROR 1707 (HY000): Found row that does not match the partition
```

可以使用IGNORE关键字，但是没有任何效果：

```sql
mysql> ALTER IGNORE TABLE e EXCHANGE PARTITION p0 WITH TABLE e2;
ERROR 1707 (HY000): Found row that does not match the partition
```

仅仅使用without validation选项才可以执行成功：

```sql
mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2 WITHOUT VALIDATION;
Query OK, 0 rows affected (0.02 sec)
```

当与表交换一个分区而其中包含有不匹配分区定义的行时，就需要管理员去修复不匹配的行，可以用`repair table or alter table ... repair partition`来实现。

### 不用逐行验证的交换分区

可以设置不用逐行验证进行分区交换，使用`without validation`在`alter table ... exchange partition`语句中，下面是验证与不验证的区别，总共有1w行的记录，有`with validation`的操花了0.74秒，有`without validation`的操作花费0.01秒：

```sql
mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e3 WITHOUT VALIDATION;
Query OK, 0 rows affected (0.01 sec)

mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2 WITH VALIDATION;
Query OK, 0 rows affected (0.74 sec)
```

### 交换子分区与未分区表

也可以使用`alter table ... exchange partition`语句进行子分区交换，如下所示用RANGE进行分区，用KEY进行子分区：

```sql
mysql> CREATE TABLE es (
    -> id INT NOT NULL,
    -> fname VARCHAR(30),
    -> lname VARCHAR(30)
    -> )
-> PARTITION BY RANGE (id)
    -> SUBPARTITION BY KEY (lname)
    -> SUBPARTITIONS 2 (
        -> PARTITION p0 VALUES LESS THAN (50),
        -> PARTITION p1 VALUES LESS THAN (100),
        -> PARTITION p2 VALUES LESS THAN (150),
        -> PARTITION p3 VALUES LESS THAN (MAXVALUE)
        -> );
Query OK, 0 rows affected (2.76 sec)
mysql> INSERT INTO es VALUES
    -> (1669, "Jim", "Smith"),
    -> (337, "Mary", "Jones"),
    -> (16, "Frank", "White"),
    -> (2005, "Linda", "Black");
Query OK, 4 rows affected (0.04 sec)
Records: 4 Duplicates: 0 Warnings: 0
mysql> CREATE TABLE es2 LIKE es;
Query OK, 0 rows affected (1.27 sec)
mysql> ALTER TABLE es2 REMOVE PARTITIONING;
Query OK, 0 rows affected (0.70 sec)
Records: 0 Duplicates: 0 Warnings: 0
```

虽然我们没有明确指定子分区的名字，但我们可以通过`subpartition_name`从information_schema中获得：

```sql
mysql> SELECT PARTITION_NAME, SUBPARTITION_NAME, TABLE_ROWS
-> FROM INFORMATION_SCHEMA.PARTITIONS
-> WHERE TABLE_NAME = 'es';
+----------------+-------------------+------------+
| PARTITION_NAME | SUBPARTITION_NAME | TABLE_ROWS |
+----------------+-------------------+------------+
| p0 | p0sp0 | 1 |
| p0 | p0sp1 | 0 |
| p1 | p1sp0 | 0 |
| p1 | p1sp1 | 0 |
| p2 | p2sp0 | 0 |
| p2 | p2sp1 | 0 |
| p3 | p3sp0 | 3 |
| p3 | p3sp1 | 0 |
+----------------+-------------------+------------+
8 rows in set (0.00 sec)
```

交换子分区p3sp0：

```sql
mysql> ALTER TABLE es EXCHANGE PARTITION p3sp0 WITH TABLE es2;
Query OK, 0 rows affected (0.29 sec)
```

结果如下所示：

```sql
mysql> SELECT PARTITION_NAME, SUBPARTITION_NAME, TABLE_ROWS
-> FROM INFORMATION_SCHEMA.PARTITIONS
-> WHERE TABLE_NAME = 'es';
+----------------+-------------------+------------+
| PARTITION_NAME | SUBPARTITION_NAME | TABLE_ROWS |
+----------------+-------------------+------------+
| p0 | p0sp0 | 1 |
| p0 | p0sp1 | 0 |
| p1 | p1sp0 | 0 |
| p1 | p1sp1 | 0 |
| p2 | p2sp0 | 0 |
| p2 | p2sp1 | 0 |
| p3 | p3sp0 | 0 |
| p3 | p3sp1 | 0 |
+----------------+-------------------+------------+
8 rows in set (0.00 sec)
mysql> SELECT * FROM es2;
+------+-------+-------+
| id | fname | lname |
+------+-------+-------+
| 1669 | Jim | Smith |
| 337 | Mary | Jones |
| 2005 | Linda | Black |
+------+-------+-------+
3 rows in set (0.00 sec)
```

## 分区的维护

分区表的维护可以通用`CHECK TABLE,OPTIMIZE TABLE,ANALYZE TABLE`和`REPAIR TABLE`完成，这些都支持分区表。你可以用`ALTER TABLE`的扩展来直接执行这些操作，如下所示：

* 重建分区：这与删除分区中的记录再重新插入是一样的效果，对碎片整理很有用，例如

    ```sql
    ALTER TABLE t1 REBUILD PARTITION p0, p1;
    ```

* 优化分区：如果你删除了一个分区的大部分行或对变长的行进行了多次修改，则你可以用`alter table ... optimize partition`重新声明任何未使用的空间和整理分区碎片，如

    ```sql
    ALTER TABLE t1 OPTIMIZE PARTITION p0, p1;
    ```

    使用`OPTIMIZE PARTITION`在特定的分区上等于运行` CHECK PARTITION,ANALYZE PARTITION, and REPAIR PARTITION`在该分区上。一些存储引擎，包括InnoDB，不支持分区优化，它会分析并重建整体表，造成一定的问题，可以使用`ALTER TABLE ... REBUILD PARTITION and ALTER TABLE ... ANALYZE PARTITION`来代替。

* 分析分区：读取或存储分区的key分布

    ```sql
    ALTER TABLE t1 ANALYZE PARTITION p3;
    ```

* 修复分区：修复坏掉的分区

    ```sql
    ALTER TABLE t1 REPAIR PARTITION p0,p1;
    ```

    正常情况下当分区包含有重复的key时，修复分区会失败。在MySQL 5.7.2及后续版本，你可以使用`alter ignore table`选项，这样所有由于重复的key而不能移动的行都会从分区删除。

* 检查分区：你可以检索分区错误与使用`check table`检查未分区表一样

    ```sql
    ALTER TABLE trb3 CHECK PARTITION p1;
    ```

    这个命令告诉你表t1的p1分区的数据或索引是否毁坏了，如果是则使用`alter table ... repair aprtition`修复该分区。`alter table ... truncate partition`用于删除一个或多个分区里的所有行，`alter table ... truncate partition all`则删除所有的分区。


## 获取分区的有关信息

有以下方法获取分区的信息：

* 使用`show create table`语句去查看在创建表时的分区信息

    ```sql
    mysql> SHOW CREATE TABLE trb3\G
    *************************** 1. row ***************************
    Table: trb3
    Create Table: CREATE TABLE `trb3` (
        `id` int(11) default NULL,
        `name` varchar(50) default NULL,
        `purchased` date default NULL
    ) ENGINE=MyISAM DEFAULT CHARSET=latin1
    PARTITION BY RANGE (YEAR(purchased)) (
        PARTITION p0 VALUES LESS THAN (1990) ENGINE = MyISAM,
        PARTITION p1 VALUES LESS THAN (1995) ENGINE = MyISAM,
        PARTITION p2 VALUES LESS THAN (2000) ENGINE = MyISAM,
        PARTITION p3 VALUES LESS THAN (2005) ENGINE = MyISAM
    )
    1 row in set (0.00 sec)
    ```

* 使用`show table status`语句查看一个表是否已分区
* 查询`INFORMATION_SCHEMA.PARTITIONS表
* 使用`explain partitions select`查看哪一个分区用在了select语句上

    ```sql
    mysql> EXPLAIN PARTITIONS SELECT * FROM trb1\G
    *************************** 1. row ***************************
    id: 1
    select_type: SIMPLE
    table: trb1
    partitions: p0,p1,p2,p3
    type: ALL
    possible_keys: NULL
    key: NULL
    key_len: NULL
    ref: NULL
    rows: 10
    Extra: Using filesort
    ```
    但`explain partitions`有以下限制：

    * 你不能同时在explain ... select 语句中使用partitions和extended关键字，否则会有语法错误
    * 如果在一个未分区表上用`explain partitions`检查查询，不会有错误发生，但分区表的值都是NULL

