分区有以下优点：

* 分区使在一个表存储更多的数据成为可能
* 来自一个分区的无用的数据可以方便的通过删除一个分区而删除它，相对来说，添加一个或新的分区来存在指定的数据也是非常方便的
* 存储在一个或多个分区的数据可以通过指定满足条件的查询来优化性能，因为它可以自动跳过其他分区的搜索
* 涉及到聚合函数的查询可以很容易的并行进行，这里的并行是说查询可以同时运行在每个分区上，最终在其结果上求和得到所有分区的数据
* 借助在多个磁盘上传播数据可以实现更大的查询吞吐量


* RANGE 分区

    > 基于属于一个给定连续区间的列值，把多行分配给分区。

* LIST 分区

    > 类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择。

* HASH分区

    > 基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包>含MySQL中有效的、产生非负整数值的任何表达式。

* KEY分区

    > 类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含>整数值。

## RANGE分区

该分区包含给定范围的一些行，这个范围应该是连续的但不重叠，使用`values less than`操作符定义的。为了接下来的几个例子，假如创建了下面的表：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

该表没有设计主键及索引，最后一个分区用小于最大值来定义，有时你也可以用基于日期列的一个表达式来分区，如：`YEAR(separtated)`：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY RANGE ( YEAR(separated) ) (
    PARTITION p0 VALUES LESS THAN (1991),
    PARTITION p1 VALUES LESS THAN (1996),
    PARTITION p2 VALUES LESS THAN (2001),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

也可能通过基于timestamp类型的列，使用unix_timestamp()来分区：

```sql
CREATE TABLE quarterly_report_status (
    report_id INT NOT NULL,
    report_status VARCHAR(20) NOT NULL,
    report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
)
PARTITION BY RANGE ( UNIX_TIMESTAMP(report_updated) ) (
    PARTITION p0 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-01-01 00:00:00') ),
    PARTITION p1 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-04-01 00:00:00') ),
    PARTITION p2 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-07-01 00:00:00') ),
    PARTITION p3 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-10-01 00:00:00') ),
    PARTITION p4 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-01-01 00:00:00') ),
    PARTITION p5 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-04-01 00:00:00') ),
    PARTITION p6 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-07-01 00:00:00') ),
    PARTITION p7 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-10-01 00:00:00') ),
    PARTITION p8 VALUES LESS THAN ( UNIX_TIMESTAMP('2010-01-01 00:00:00') ),
    PARTITION p9 VALUES LESS THAN (MAXVALUE)
);
```

Range分区很有用在以下条件下：

* 当你想要删除旧数据时。你可以使用`alter table employees drop partition p0`来删除很早的1991年的数据
* 当你要使用包含日期或时间值，或递增的值的列时。
* 你经常运行直接依赖于用于分割表的列的查询。例如，当执行像`explain partitions select count(*) from employees where separated between '2000-01-01' and '2000-12-31' group by store_id`，MySQL会快速决定只需要扫描p2分区。

根据时间间隔分区方案：

1. 通过RANGE来分区表，为分区表达式使用一个函数操作符在DATE,TIME,DATETIME列上，并返回一个整形值

    ```sql
    CREATE TABLE members (
        firstname VARCHAR(25) NOT NULL,
        lastname VARCHAR(25) NOT NULL,
        username VARCHAR(16) NOT NULL,
        email VARCHAR(35),
        joined DATE NOT NULL
    )
    PARTITION BY RANGE( YEAR(joined) ) (
        PARTITION p0 VALUES LESS THAN (1960),
        PARTITION p1 VALUES LESS THAN (1970),
        PARTITION p2 VALUES LESS THAN (1980),
        PARTITION p3 VALUES LESS THAN (1990),
        PARTITION p4 VALUES LESS THAN MAXVALUE
    );
    ```

    也可用unix_timestamp()函数在timestamp列上：

    ```sql
    CREATE TABLE quarterly_report_status (
        report_id INT NOT NULL,
        report_status VARCHAR(20) NOT NULL,
        report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
        )
    PARTITION BY RANGE ( UNIX_TIMESTAMP(report_updated) ) (
        PARTITION p0 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-01-01 00:00:00') ),
        PARTITION p1 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-04-01 00:00:00') ),
        PARTITION p2 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-07-01 00:00:00') ),
        PARTITION p3 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-10-01 00:00:00') ),
        PARTITION p4 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-01-01 00:00:00') ),
        PARTITION p5 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-04-01 00:00:00') ),
        PARTITION p6 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-07-01 00:00:00') ),
        PARTITION p7 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-10-01 00:00:00') ),
        PARTITION p8 VALUES LESS THAN ( UNIX_TIMESTAMP('2010-01-01 00:00:00') ),
        PARTITION p9 VALUES LESS THAN (MAXVALUE)
        );
    ```


2. 通过RANGE COLUMNS分区表，使用一个DATE或DATETIME列作为分区列：

    ```sql
    CREATE TABLE members (
        firstname VARCHAR(25) NOT NULL,
        lastname VARCHAR(25) NOT NULL,
        username VARCHAR(16) NOT NULL,
        email VARCHAR(35),
        joined DATE NOT NULL
    )
    PARTITION BY RANGE COLUMNS(joined) (
        PARTITION p0 VALUES LESS THAN ('1960-01-01'),
        PARTITION p1 VALUES LESS THAN ('1970-01-01'),
        PARTITION p2 VALUES LESS THAN ('1980-01-01'),
        PARTITION p3 VALUES LESS THAN ('1990-01-01'),
        PARTITION p4 VALUES LESS THAN MAXVALUE
    );
    ```

## LIST分区

list分区在很多方面与range分区类似，在range分区里，每个分区都必须明确定义。这两种类型的分区主要的区别就是，在list分区里，每个分区的定义与选择是基于一组值列表里的列值成员，而不是一组连续范围的值。这是通过`PARTITION BY LIST(expr)`完成的，而expr是一个列值或一个基于列值的返回整形值的表达式，然后通过`VALUES IN (value_list(的方式定义，而value_list是以逗号分隔的整数。与range分区不同，list分区不需要以特定的顺序定义。

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY LIST(store_id) (
    PARTITION pNorth VALUES IN (3,5,6,9,17),
    PARTITION pEast VALUES IN (1,2,10,11,19,20),
    PARTITION pWest VALUES IN (4,12,13,14,18),
    PARTITION pCentral VALUES IN (7,8,15,16)
);
```

这样定义会很方便的添加或删除指定区域的员工，如假设西部地区的商店卖给了另一个国家，则通过`alter table employees truncate partition pWest`便可删除该区域的商店，使用`alter table employees drop partition pWest`也可以删除所有的行，但是也会删除表中对该分区的定义，你还需要用`alter table ... add partition`语句来重新定义

所有list分区的表达式应该用`PARTITION ... VALUES IN (...)`语句来覆盖，一个包含没有匹配分区的列值的INSERT语句会出错：

```sql
mysql> CREATE TABLE h2 (
    -> c1 INT,
    -> c2 INT
    -> )
    -> PARTITION BY LIST(c1) (
    -> PARTITION p0 VALUES IN (1, 4, 7),
    -> PARTITION p1 VALUES IN (2, 5, 8)
    -> );
Query OK, 0 rows affected (0.11 sec)
mysql> INSERT INTO h2 VALUES (3, 5);
ERROR 1525 (HY000): Table has no partition for value 3
```

一条插入多组值的insert语句，如果用的是InnoDB引擎，则只要有一个不匹配相应的分区，则都会插入失败。MyISAM引擎表则会部分插入失败。你可以用`IGNORE`关键字来忽略错误，如果有不匹配分区的行，则不进行插入，但其他的匹配的还可以插入：

```sql
mysql> TRUNCATE h2;
Query OK, 1 row affected (0.00 sec)
mysql> SELECT * FROM h2;
Empty set (0.00 sec)
mysql> INSERT IGNORE INTO h2 VALUES (2, 5), (6, 10), (7, 5), (3, 1), (1, 9);
Query OK, 3 rows affected (0.00 sec)
Records: 5 Duplicates: 2 Warnings: 0
mysql> SELECT * FROM h2;
+------+------+
| c1 | c2 |
+------+------+
| 7 | 5 |
| 1 | 9 |
| 2 | 5 |
+------+------+
3 rows in set (0.00 sec)
```

## COLUMNS分区

该分区是range和list分区的进化，它可以在分区keys中使用多列。另外`RANGE COLUMNS`分区和`LIST COLUMNS`分区支持用非整型列来定义ranges或list成员的值。以下列表说明了哪些类型可以用于该分区：

* 所有的整形：tinyint,smallint,mediumint,int(integer),bigint，其他的数据类型如decimal,float等不支持用来分区
* date和datetime,使用其他与日期或时间相关的列不能作为分区列
* 字符类型：char,varchar,binary,varbinary，而text和blog列不能作为分区列

### RANGE COLUMNS分区

RANGE COLUMNS分区类似于range分区，但它可以让你使用ranges基于多列值定义分区。与RANGE分区的不同之处在以下几点：

* RANGE COLUMNS不接受表达式，仅仅是列名
* RANGE COLUMNS接受由一列或多列组成的一个列表
* RANGE COLUMNS分区是基于无组之间的比较，而非标值之间的比较
* RANGE COLUMNS分区列不限于整形列，string,date,datetime列也可以用于分区列

用RANGE COLUMNS创建分区的语法如下所示：

```sql
CREATE TABLE table_name
PARTITIONED BY RANGE COLUMNS(column_list) (
    PARTITION partition_name VALUES LESS THAN (value_list)[,
    PARTITION partition_name VALUES LESS THAN (value_list)][,
    ...]
)
column_list:
    column_name[, column_name][, ...]
value_list:
    value[, value][, ...]
```

column_list有多少列，value_list必须有相同数量的值，也就是说，如果你使用N列在COLUMNS语句中，则每个VALUES LESS THAN语句也必须是有Ｎ个值的列表，且顺序、类型都必须一致：

```sql
mysql> CREATE TABLE rcx (
-> a INT,
-> b INT,
-> c CHAR(3),
-> d INT
-> )
-> PARTITION BY RANGE COLUMNS(a,d,c) (
    -> PARTITION p0 VALUES LESS THAN (5,10,'ggg'),
    -> PARTITION p1 VALUES LESS THAN (10,20,'mmmm'),
    -> PARTITION p2 VALUES LESS THAN (15,30,'sss'),
    -> PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE,MAXVALUE)
-> );
Query OK, 0 rows affected (0.15 sec)
```

而RANGE分区表的创建则如下所示：

```sql
CREATE TABLE r1 (
    a INT,
    b INT
)
PARTITION BY RANGE (a) (
    PARTITION p0 VALUES LESS THAN (5),
    PARTITION p1 VALUES LESS THAN (MAXVALUE)
);
```
则以下的三条插入语句都是插入到p1分区的，因为a列都大于５，可以查询INFORMATION_SCHEMA.PARTITIONS表看结果：

```sql
mysql> INSERT INTO r1 VALUES (5,10), (5,11), (5,12);
Query OK, 3 rows affected (0.00 sec)
Records: 3 Duplicates: 0 Warnings: 0
mysql> SELECT PARTITION_NAME,TABLE_ROWS
    -> FROM INFORMATION_SCHEMA.PARTITIONS
    -> WHERE TABLE_NAME = 'r1';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0 | 0 |
| p1 | 3 |
+----------------+------------+
2 rows in set (0.00 sec)
```

现在我们用RANGE COLUMNS分区表通过a,b列，如：

```sql
CREATE TABLE rc1 (
    a INT,
    b INT
)
PARTITION BY RANGE COLUMNS(a, b) (
    PARTITION p0 VALUES LESS THAN (5, 12),
    PARTITION p3 VALUES LESS THAN (MAXVALUE, MAXVALUE)
);
```

如果我们插入同样的行到表中，则都会插入p0分区中，就会有不一样的效果：

```sql
mysql> INSERT INTO rc1 VALUES (5,10), (5,11), (5,12);
Query OK, 3 rows affected (0.00 sec)
Records: 3 Duplicates: 0 Warnings: 0
mysql> SELECT PARTITION_NAME,TABLE_ROWS
    -> FROM INFORMATION_SCHEMA.PARTITIONS
    -> WHERE TABLE_NAME = 'rc1';
+--------------+----------------+------------+
| TABLE_SCHEMA | PARTITION_NAME | TABLE_ROWS |
+--------------+----------------+------------+
| p | p0 | 2 |
| p | p1 | 1 |
+--------------+----------------+------------+
2 rows in set (0.00 sec)
```

如果通过RANGE COLUMNS使用仅一单独的列进行分区，则与RANGE的效果是一样的，也可以通过RANGE COLUMNS限制其中的一列或多列的值，其他列值是一样的，来进行分区，如：

```sql
CREATE TABLE rc2 (
    a INT,
    b INT
)
PARTITION BY RANGE COLUMNS(a,b) (
    PARTITION p0 VALUES LESS THAN (0,10),
    PARTITION p1 VALUES LESS THAN (10,20),
    PARTITION p2 VALUES LESS THAN (10,30),
    PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE)
);
CREATE TABLE rc3 (
    a INT,
    b INT
)
PARTITION BY RANGE COLUMNS(a,b) (
    PARTITION p0 VALUES LESS THAN (0,10),
    PARTITION p1 VALUES LESS THAN (10,20),
    PARTITION p2 VALUES LESS THAN (10,30),
    PARTITION p3 VALUES LESS THAN (10,35),
    PARTITION p4 VALUES LESS THAN (20,40),
    PARTITION p5 VALUES LESS THAN (MAXVALUE,MAXVALUE)
);

REATE TABLE rc4 (
    a INT,
    b INT,
    c INT
)
PARTITION BY RANGE COLUMNS(a,b,c) (
    PARTITION p0 VALUES LESS THAN (0,25,50),
    PARTITION p1 VALUES LESS THAN (10,20,100),
    PARTITION p2 VALUES LESS THAN (10,30,50)
    PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE,MAXVALUE)
);
```

如前面所说也可以用非整形列作为分区列进行分区，如：

```sql
CREATE TABLE employees_by_lname (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE COLUMNS (lname) (
    PARTITION p0 VALUES LESS THAN ('g'),
    PARTITION p1 VALUES LESS THAN ('m'),
    PARTITION p2 VALUES LESS THAN ('t'),
    PARTITION p3 VALUES LESS THAN (MAXVALUE)
);
```

也可以通过修改之前的表来实现：

```sql
ALTER TABLE employees PARTITION BY RANGE COLUMNS (lname) (
    PARTITION p0 VALUES LESS THAN ('g'),
    PARTITION p1 VALUES LESS THAN ('m'),
    PARTITION p2 VALUES LESS THAN ('t'),
    PARTITION p3 VALUES LESS THAN (MAXVALUE)
);
```

### LIST COLUMNS分区

MySQL 5.7对LIST COLUMNS分区提供了支持，这是LIST分区的进化，可以使用多列做为分区keys且也可以不是整形的列，你可以使用字符串型，date,datetime型的列。可以根据城市名来分区：

```sql
CREATE TABLE customers_1 (
    first_name VARCHAR(25),
    last_name VARCHAR(25),
    street_1 VARCHAR(30),
    street_2 VARCHAR(30),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY LIST COLUMNS(city) (
    PARTITION pRegion_1 VALUES IN('Oskarshamn', 'Högsby', 'Mönsterås'),
    PARTITION pRegion_2 VALUES IN('Vimmerby', 'Hultsfred', 'Västervik'),
    PARTITION pRegion_3 VALUES IN('Nässjö', 'Eksjö', 'Vetlanda'),
    PARTITION pRegion_4 VALUES IN('Uppvidinge', 'Alvesta', 'Växjo')
);
```

通过RANGE COLUMNS分区，你可以不用使用表达式在COLUMNS()语句中去转化列值为整型，也可以用DATE和DATETIME列作为LIST COLUMNS分区列，如：

```sql
CREATE TABLE customers_2 (
    first_name VARCHAR(25),
    last_name VARCHAR(25),
    street_1 VARCHAR(30),
    street_2 VARCHAR(30),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY LIST COLUMNS(renewal) (
    PARTITION pWeek_1 VALUES IN('2010-02-01', '2010-02-02', '2010-02-03',
        '2010-02-04', '2010-02-05', '2010-02-06', '2010-02-07'),
    PARTITION pWeek_2 VALUES IN('2010-02-08', '2010-02-09', '2010-02-10',
        '2010-02-11', '2010-02-12', '2010-02-13', '2010-02-14'),
    PARTITION pWeek_3 VALUES IN('2010-02-15', '2010-02-16', '2010-02-17',
        '2010-02-18', '2010-02-19', '2010-02-20', '2010-02-21'),
    PARTITION pWeek_4 VALUES IN('2010-02-22', '2010-02-23', '2010-02-24',
        '2010-02-25', '2010-02-26', '2010-02-27', '2010-02-28')
);
```

但这样维护日期值会很多很大，但用RANGE或RANGE COLUMNS分区代替会更好，如：

```sql
REATE TABLE customers_3 (
    first_name VARCHAR(25),
    last_name VARCHAR(25),
    street_1 VARCHAR(30),
    street_2 VARCHAR(30),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY RANGE COLUMNS(renewal) (
    PARTITION pWeek_1 VALUES LESS THAN('2010-02-09'),
    PARTITION pWeek_2 VALUES LESS THAN('2010-02-15'),
    PARTITION pWeek_3 VALUES LESS THAN('2010-02-22'),
    PARTITION pWeek_4 VALUES LESS THAN('2010-03-01')
);
```


## HASH分区

HASH分区多用于在多个分区之间均匀的分配数据，需要指定一个列值或基于一个列值的表达式进行散列和分区表要分区的数量。用HASH分区进行分区表，则需要在create table语句后面追加PARTITION BY HASH(expr)语句，且expr是一个返回整型值的表达式，同时还有PARTITIONS num语句，num指定一个要分多少区的正数值，如果没有指定该值，则默认值为1，针对expr的表达式，记录会存储到分区N中，则N = MOD(expr, num)：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY HASH(store_id)
PARTITIONS 4;
```

> 这里如果一个分区表有唯一索引key，则作为hash分区key的columns_list必须是唯一索引的一部分


### LINEAR HASH分区

需要添加LINEAR关键字在PARTITION BY语句中，如：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY LINEAR HASH( YEAR(hired) )
PARTITIONS 4;
```

则分区数N的算法如下所示：

1. V = POWER(2, CEILING(LOG(2, num)))
2. Set N = F(column_list) & (V - 1)
3. While N >= num:
    
    * Set V = CEIL(V / 2)
    * Set N = N & (V - 1)

## KEY分区

key分区和hash分区相似，不同在于hash分区是用户自定义函数进行分区，key分区使用mysql数据库提供的函数进行分区，NDB cluster使用MD5函数来分区，对于其他存储引擎mysql使用内部的hash函数，这些函数基于password()一样的算法。

语法是create table ... partition by key语句，与HASH分区主要不同有：

* 使用KEY关键字而非HASH
* KEY()使用0或多列的名字，每个用于key的列都必须是表主键的一部分，如果没有指定列，则默认用表的主键做为key，如果没有主键，有唯一索引，则唯一索引用于key如：

    ```sql
    CREATE TABLE k1 (
        id INT NOT NULL,
        name VARCHAR(20),
        UNIQUE KEY (id)
    )
    PARTITION BY KEY()
    PARTITIONS 2;
    ```
如果唯一索引没有被定义为NOT NULL，则上面语句将出错。

不像其他分区类型，作为key的分区列可以不限于整型或NULL值，如：

```sql
CREATE TABLE tm1 (
    s1 CHAR(32) PRIMARY KEY
)
PARTITION BY KEY(s1)
PARTITIONS 10;
```

## 子分区

子分区也被叫作是组合分区，针对每个分区表进一步分区，如：

```sql
CREATE TABLE ts (id INT, purchased DATE)
    PARTITION BY RANGE( YEAR(purchased) )
    SUBPARTITION BY HASH( TO_DAYS(purchased) )
    SUBPARTITIONS 2 (
        PARTITION p0 VALUES LESS THAN (1990),
        PARTITION p1 VALUES LESS THAN (2000),
        PARTITION p2 VALUES LESS THAN MAXVALUE
    );
```

表ts有三个分区，每个分区又分为2个子分区，事实上是3*2=6个分区。也可以明确用SUBPARTITION语句指定选项对个别分区，如：

```sql
CREATE TABLE ts (id INT, purchased DATE)
    PARTITION BY RANGE( YEAR(purchased) )
    SUBPARTITION BY HASH( TO_DAYS(purchased) ) (
        PARTITION p0 VALUES LESS THAN (1990) (
            SUBPARTITION s0,
            SUBPARTITION s1
            ),
        PARTITION p1 VALUES LESS THAN (2000) (
            SUBPARTITION s2,
            SUBPARTITION s3
            ),
        PARTITION p2 VALUES LESS THAN MAXVALUE (
            SUBPARTITION s4,
            SUBPARTITION s5
            )
        );
```

以下是要关注的语法点：

* 每个分区必须有相同数量的子分区
* 如果你用SUBPARTITION明确指定子分区在一个分区表的任何分区上，则，你必须定义他们，换句话说，下面的语句将失败：

    ```sql
    CREATE TABLE ts (id INT, purchased DATE)
    PARTITION BY RANGE( YEAR(purchased) )
    SUBPARTITION BY HASH( TO_DAYS(purchased) ) (
        PARTITION p0 VALUES LESS THAN (1990) (
            SUBPARTITION s0,
            SUBPARTITION s1
            ),
        PARTITION p1 VALUES LESS THAN (2000),
        PARTITION p2 VALUES LESS THAN MAXVALUE (
            SUBPARTITION s2,
            SUBPARTITION s3
            )
        );
    ```

* 每一个SUBPARTITION语句必须包含一个名字为子分区
* 子分区名必须在整个表中是唯一的，如：

   ```sql
   CREATE TABLE ts (id INT, purchased DATE)
    PARTITION BY RANGE( YEAR(purchased) )
    SUBPARTITION BY HASH( TO_DAYS(purchased) ) (
        PARTITION p0 VALUES LESS THAN (1990) (
            SUBPARTITION s0,
            SUBPARTITION s1
            ),
        PARTITION p1 VALUES LESS THAN (2000) (
            SUBPARTITION s2,
            SUBPARTITION s3
            ),
        PARTITION p2 VALUES LESS THAN MAXVALUE (
            SUBPARTITION s4,
            SUBPARTITION s5
            )
        );
    ```

子分区也可以用于特别是大表去健在数据和索引到多个磁盘上，如：

```sql
CREATE TABLE ts (id INT, purchased DATE)
    PARTITION BY RANGE(YEAR(purchased))
    SUBPARTITION BY HASH( TO_DAYS(purchased) ) (
        PARTITION p0 VALUES LESS THAN (1990) (
            SUBPARTITION s0a
            DATA DIRECTORY = '/disk0'
            INDEX DIRECTORY = '/disk1',
            SUBPARTITION s0b
            DATA DIRECTORY = '/disk2'
            INDEX DIRECTORY = '/disk3'
            ),
        PARTITION p1 VALUES LESS THAN (2000) (
            SUBPARTITION s1a
            DATA DIRECTORY = '/disk4/data'
            INDEX DIRECTORY = '/disk4/idx',
            SUBPARTITION s1b
            DATA DIRECTORY = '/disk5/data'
            INDEX DIRECTORY = '/disk5/idx'
            ),
        PARTITION p2 VALUES LESS THAN MAXVALUE (
            SUBPARTITION s2a,
            SUBPARTITION s2b
            )
        );
```

## MySQL分区是如何处理NULL值的

MySQL分区确实没有什么不允许NULL作为一个分区表达式的值的，但不同的分区类型针对NULL的行为可能会有所不同。

### RANGE分区对NULL值的处理

如果你插入一行到RANGE分区表中，且决定分区的列值是NULL，则该行被插入到最低的分区里

首先创建相应的分区表：

```sql
mysql> CREATE TABLE t1 (
    -> c1 INT,
    -> c2 VARCHAR(20)
    -> )
    -> PARTITION BY RANGE(c1) (
    -> PARTITION p0 VALUES LESS THAN (0),
    -> PARTITION p1 VALUES LESS THAN (10),
    -> PARTITION p2 VALUES LESS THAN MAXVALUE
    -> );
Query OK, 0 rows affected (0.09 sec)
mysql> CREATE TABLE t2 (
    -> c1 INT,
    -> c2 VARCHAR(20)
    -> )
    -> PARTITION BY RANGE(c1) (
    -> PARTITION p0 VALUES LESS THAN (-5),
    -> PARTITION p1 VALUES LESS THAN (0),
    -> PARTITION p2 VALUES LESS THAN (10),
    -> PARTITION p3 VALUES LESS THAN MAXVALUE
    -> );
Query OK, 0 rows affected (0.09 sec)
```

其次查看分区情况：

```sql
mysql> SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
> FROM INFORMATION_SCHEMA.PARTITIONS
> WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME LIKE 't_';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
| t1 | p0 | 0 | 0 | 0 |
| t1 | p1 | 0 | 0 | 0 |
| t1 | p2 | 0 | 0 | 0 |
| t2 | p0 | 0 | 0 | 0 |
| t2 | p1 | 0 | 0 | 0 |
| t2 | p2 | 0 | 0 | 0 |
| t2 | p3 | 0 | 0 | 0 |
+------------+----------------+------------+----------------+-------------+
7 rows in set (0.00 sec)
```

再插入有NULL值的行：

```sql
mysql> INSERT INTO t1 VALUES (NULL, 'mothra');
Query OK, 1 row affected (0.00 sec)
mysql> INSERT INTO t2 VALUES (NULL, 'mothra');
Query OK, 1 row affected (0.00 sec)
mysql> SELECT * FROM t1;
+------+--------+
| id | name |
+------+--------+
| NULL | mothra |
+------+--------+
1 row in set (0.00 sec)
mysql> SELECT * FROM t2;
+------+--------+
| id | name |
+------+--------+
| NULL | mothra |
+------+--------+
1 row in set (0.00 sec)
```

最后查看分区结果INFORMATION_SCHEMA.PARTITIONS，带有NULL值的行已插入到最低分区：

```sql
mysql> SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
> FROM INFORMATION_SCHEMA.PARTITIONS
> WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME LIKE 't_';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
| t1 | p0 | 1 | 20 | 20 |
| t1 | p1 | 0 | 0 | 0 |
| t1 | p2 | 0 | 0 | 0 |
| t2 | p0 | 1 | 20 | 20 |
| t2 | p1 | 0 | 0 | 0 |
| t2 | p2 | 0 | 0 | 0 |
| t2 | p3 | 0 | 0 | 0 |
+------------+----------------+------------+----------------+-------------+
7 rows in set (0.01 sec)
```

你可以通过丢弃存储在每个表最低分区中的行还证明NULL值是存储在最低分区的：

```sql
mysql> ALTER TABLE t1 DROP PARTITION p0;
Query OK, 0 rows affected (0.16 sec)
mysql> ALTER TABLE t2 DROP PARTITION p0;
Query OK, 0 rows affected (0.16 sec)
mysql> SELECT * FROM t1;
Empty set (0.00 sec)
mysql> SELECT * FROM t2;
Empty set (0.00 sec)
```

NULL值的处理在使用SQL函数做为分区表达式时也是一样的,YEAR(NULL) 返回NULL：

```sql
CREATE TABLE tndate (
    id INT,
    dt DATE
)
PARTITION BY RANGE( YEAR(dt) ) (
    PARTITION p0 VALUES LESS THAN (1990),
    PARTITION p1 VALUES LESS THAN (2000),
    PARTITION p2 VALUES LESS THAN MAXVALUE
);
```

### LIST分区对NULL值的处理

LIST分区是承认NULL值的，仅当分区列表中包含NULL值，相反，如果没有明确使用NULL值作为分区，则是拒绝插入NULL值的。

没有明确指定NULL值分区的：

```sql
mysql> CREATE TABLE ts1 (
    -> c1 INT,
    -> c2 VARCHAR(20)
    -> )
    -> PARTITION BY LIST(c1) (
    -> PARTITION p0 VALUES IN (0, 3, 6),
    -> PARTITION p1 VALUES IN (1, 4, 7),
    -> PARTITION p2 VALUES IN (2, 5, 8)
    -> );
Query OK, 0 rows affected (0.01 sec)
mysql> INSERT INTO ts1 VALUES (9, 'mothra');
ERROR 1504 (HY000): Table has no partition for value 9
mysql> INSERT INTO ts1 VALUES (NULL, 'mothra');
ERROR 1504 (HY000): Table has no partition for value NULL
```

有明确NULL值分区的：

```sql
mysql> CREATE TABLE ts2 (
    -> c1 INT,
    -> c2 VARCHAR(20)
    -> )
-> PARTITION BY LIST(c1) (
    -> PARTITION p0 VALUES IN (0, 3, 6),
    -> PARTITION p1 VALUES IN (1, 4, 7),
    -> PARTITION p2 VALUES IN (2, 5, 8),
    -> PARTITION p3 VALUES IN (NULL)
    -> );
Query OK, 0 rows affected (0.01 sec)
mysql> CREATE TABLE ts3 (
    -> c1 INT,
    -> c2 VARCHAR(20)
    -> )
-> PARTITION BY LIST(c1) (
    -> PARTITION p0 VALUES IN (0, 3, 6),
    -> PARTITION p1 VALUES IN (1, 4, 7, NULL),
    -> PARTITION p2 VALUES IN (2, 5, 8)
    -> );
Query OK, 0 rows affected (0.01 sec)
```

插入结果如下：

```sql
mysql> SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
> FROM INFORMATION_SCHEMA.PARTITIONS
> WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME LIKE 'ts_';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
| ts2 | p0 | 0 | 0 | 0 |
| ts2 | p1 | 0 | 0 | 0 |
| ts2 | p2 | 0 | 0 | 0 |
| ts2 | p3 | 1 | 20 | 20 |
| ts3 | p0 | 0 | 0 | 0 |
| ts3 | p1 | 1 | 20 | 20 |
| ts3 | p2 | 0 | 0 | 0 |
+------------+----------------+------------+----------------+-------------+
7 rows in set (0.01 sec)
```


### HASH和KEY分区对NULL值的处理

NULL值针对HASH和KEY分区表来说有点不同，任何的带NULL值的分区表达式都会被作为返回0来对待。

创建表且初始分区情况如下：

```sql
mysql> CREATE TABLE th (
    -> c1 INT,
    -> c2 VARCHAR(20)
    -> )
-> PARTITION BY HASH(c1)
-> PARTITIONS 2;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT TABLE_NAME,PARTITION_NAME,TABLE_ROWS,AVG_ROW_LENGTH,DATA_LENGTH
> FROM INFORMATION_SCHEMA.PARTITIONS
> WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME ='th';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
| th | p0 | 0 | 0 | 0 |
| th | p1 | 0 | 0 | 0 |
+------------+----------------+------------+----------------+-------------+
2 rows in set (0.00 sec)
```

插入带有NULL和0值列的行后，情况如下所示：

```sql
mysql> INSERT INTO th VALUES (NULL, 'mothra'), (0, 'gigan');
Query OK, 1 row affected (0.00 sec)
mysql> SELECT * FROM th;
+------+---------+
| c1 | c2 |
+------+---------+
| NULL | mothra |
+------+---------+
| 0 | gigan |
+------+---------+
2 rows in set (0.01 sec)

mysql> SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
> FROM INFORMATION_SCHEMA.PARTITIONS
> WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME ='th';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
| th | p0 | 2 | 20 | 20 |
| th | p1 | 0 | 0 | 0 |
+------------+----------------+------------+----------------+-------------+
2 rows in set (0.00 sec)
```

任何的NULL值，NULL MOD N 总是为NULL，所以NULL是被作为0来对待的。



