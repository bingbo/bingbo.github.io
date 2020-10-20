* 登录：`mysql -h fe.xxxx.com -u root -p`
* 退出：`exit|quit`
* 查询：`select now(),user(),version();` 
                
    > 注：在函数名秘括号之间不能有空格；  
        结束查询命令的另一种方法是用\g来代替分号，\G会以竖形来显示；  
        如果输入了几条命令，但不想执行它，则可以用\c来清除取消；

* 选择数据库：`use test;`或是`mysql -u root -p test;`
* 创建数据库：`create database mydb;`
* 创建表：

    ```mysql
    help create table;
    CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
        (create_definition,...)
        [table_options]
        [partition_options]

    Or:

    CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
        [(create_definition,...)]
        [table_options]
        [partition_options]
        select_statement

    Or:
    CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
        { LIKE old_tbl_name | (LIKE old_tbl_name) }
    create_definition:
        col_name column_definition
      | [CONSTRAINT [symbol]] PRIMARY KEY [index_type] (index_col_name,...)
          [index_option] ...
      | {INDEX|KEY} [index_name] [index_type] (index_col_name,...)
          [index_option] ...
      | [CONSTRAINT [symbol]] UNIQUE [INDEX|KEY]
          [index_name] [index_type] (index_col_name,...)
          [index_option] ...
      | {FULLTEXT|SPATIAL} [INDEX|KEY] [index_name] (index_col_name,...)
          [index_option] ...
      | [CONSTRAINT [symbol]] FOREIGN KEY
          [index_name] (index_col_name,...) reference_definition
      | CHECK (expr)
    column_definition:
        data_type [NOT NULL | NULL] [DEFAULT default_value]
          [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY]
          [COMMENT 'string']
          [COLUMN_FORMAT {FIXED|DYNAMIC|DEFAULT}]
          [STORAGE {DISK|MEMORY|DEFAULT}]
          [reference_definition]
    data_type:
        BIT[(length)]
      | TINYINT[(length)] [UNSIGNED] [ZEROFILL]
      | SMALLINT[(length)] [UNSIGNED] [ZEROFILL]
      | MEDIUMINT[(length)] [UNSIGNED] [ZEROFILL]
      | INT[(length)] [UNSIGNED] [ZEROFILL]
      | INTEGER[(length)] [UNSIGNED] [ZEROFILL]
      | BIGINT[(length)] [UNSIGNED] [ZEROFILL]
      | REAL[(length,decimals)] [UNSIGNED] [ZEROFILL]
      | DOUBLE[(length,decimals)] [UNSIGNED] [ZEROFILL]
      | FLOAT[(length,decimals)] [UNSIGNED] [ZEROFILL]
      | DECIMAL[(length[,decimals])] [UNSIGNED] [ZEROFILL]
      | NUMERIC[(length[,decimals])] [UNSIGNED] [ZEROFILL]
      | DATE
      | TIME
      | TIMESTAMP
      | DATETIME
      | YEAR
      | CHAR[(length)]
          [CHARACTER SET charset_name] [COLLATE collation_name]
      | VARCHAR(length)
          [CHARACTER SET charset_name] [COLLATE collation_name]
      | BINARY[(length)]
      | VARBINARY(length)
      | TINYBLOB
      | BLOB
      | MEDIUMBLOB
      | LONGBLOB
      | TINYTEXT [BINARY]
          [CHARACTER SET charset_name] [COLLATE collation_name]
      | TEXT [BINARY]
          [CHARACTER SET charset_name] [COLLATE collation_name]
      | MEDIUMTEXT [BINARY]
          [CHARACTER SET charset_name] [COLLATE collation_name]
      | LONGTEXT [BINARY]
          [CHARACTER SET charset_name] [COLLATE collation_name]
      | ENUM(value1,value2,value3,...)
          [CHARACTER SET charset_name] [COLLATE collation_name]
      | SET(value1,value2,value3,...)
          [CHARACTER SET charset_name] [COLLATE collation_name]
      | spatial_type
    index_col_name:
        col_name [(length)] [ASC | DESC]
    index_type:
        USING {BTREE | HASH | RTREE}
    index_option:
        KEY_BLOCK_SIZE [=] value
      | index_type
      | WITH PARSER parser_name
    reference_definition:
        REFERENCES tbl_name (index_col_name,...)
          [MATCH FULL | MATCH PARTIAL | MATCH SIMPLE]
          [ON DELETE reference_option]
          [ON UPDATE reference_option]
    reference_option:
        RESTRICT | CASCADE | SET NULL | NO ACTION
    table_options:
        table_option [[,] table_option] ...
    table_option:
        ENGINE [=] engine_name
      | AUTO_INCREMENT [=] value
      | AVG_ROW_LENGTH [=] value
      | [DEFAULT] CHARACTER SET [=] charset_name
      | CHECKSUM [=] {0 | 1}
      | [DEFAULT] COLLATE [=] collation_name
      | COMMENT [=] 'string'
      | CONNECTION [=] 'connect_string'
      | DATA DIRECTORY [=] 'absolute path to directory'
      | DELAY_KEY_WRITE [=] {0 | 1}
      | INDEX DIRECTORY [=] 'absolute path to directory'
      | INSERT_METHOD [=] { NO | FIRST | LAST }
      | KEY_BLOCK_SIZE [=] value
      | MAX_ROWS [=] value
      | MIN_ROWS [=] value
      | PACK_KEYS [=] {0 | 1 | DEFAULT}
      | PASSWORD [=] 'string'
      | ROW_FORMAT [=] {DEFAULT|DYNAMIC|FIXED|COMPRESSED|REDUNDANT|COMPACT}
      | TABLESPACE tablespace_name [STORAGE {DISK|MEMORY|DEFAULT}]
      | UNION [=] (tbl_name[,tbl_name]...)
    partition_options:
        PARTITION BY
            { [LINEAR] HASH(expr)
            | [LINEAR] KEY(column_list)
            | RANGE(expr)
            | LIST(expr) }
        [PARTITIONS num]
        [SUBPARTITION BY
            { [LINEAR] HASH(expr)
            | [LINEAR] KEY(column_list) }
          [SUBPARTITIONS num]
        ]
        [(partition_definition [, partition_definition] ...)]
    partition_definition:
        PARTITION partition_name
            [VALUES {LESS THAN {(expr) | MAXVALUE} | IN (value_list)}]
            [[STORAGE] ENGINE [=] engine_name]
            [COMMENT [=] 'comment_text' ]
            [DATA DIRECTORY [=] 'data_dir']
            [INDEX DIRECTORY [=] 'index_dir']
            [MAX_ROWS [=] max_number_of_rows]
            [MIN_ROWS [=] min_number_of_rows]
            [TABLESPACE [=] tablespace_name]
            [NODEGROUP [=] node_group_id]
            [(subpartition_definition [, subpartition_definition] ...)]
    subpartition_definition:
        SUBPARTITION logical_name
            [[STORAGE] ENGINE [=] engine_name]
            [COMMENT [=] 'comment_text' ]
            [DATA DIRECTORY [=] 'data_dir']
            [INDEX DIRECTORY [=] 'index_dir']
            [MAX_ROWS [=] max_number_of_rows]
            [MIN_ROWS [=] min_number_of_rows]
            [TABLESPACE [=] tablespace_name]
            [NODEGROUP [=] node_group_id]
    select_statement:
        [IGNORE | REPLACE] [AS] SELECT ... (Some legal select statement)
    ```

* 查看表结构：

    ```mysql
    desc test_product; 
    explain test_product;
    show fields from test_product;
    ```
* show语句：

    ```mysql
    SHOW AUTHORS
    SHOW {BINARY | MASTER} LOGS
    SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count]
    SHOW CHARACTER SET [like_or_where]
    SHOW COLLATION [like_or_where]
    SHOW [FULL] COLUMNS FROM tbl_name [FROM db_name] [like_or_where]
    SHOW CONTRIBUTORS
    SHOW CREATE DATABASE db_name
    SHOW CREATE EVENT event_name
    SHOW CREATE FUNCTION func_name
    SHOW CREATE PROCEDURE proc_name
    SHOW CREATE TABLE tbl_name
    SHOW CREATE TRIGGER trigger_name
    SHOW CREATE VIEW view_name
    SHOW DATABASES [like_or_where]
    SHOW ENGINE engine_name {STATUS | MUTEX}
    SHOW [STORAGE] ENGINES
    SHOW ERRORS [LIMIT [offset,] row_count]
    SHOW EVENTS
    SHOW FUNCTION CODE func_name
    SHOW FUNCTION STATUS [like_or_where]
    SHOW GRANTS FOR user
    SHOW INDEX FROM tbl_name [FROM db_name]
    SHOW INNODB STATUS
    SHOW MASTER STATUS
    SHOW OPEN TABLES [FROM db_name] [like_or_where]
    SHOW PLUGINS
    SHOW PROCEDURE CODE proc_name
    SHOW PROCEDURE STATUS [like_or_where]
    SHOW PRIVILEGES
    SHOW [FULL] PROCESSLIST
    SHOW PROFILE [types] [FOR QUERY n] [OFFSET n] [LIMIT n]
    SHOW PROFILES
    SHOW SCHEDULER STATUS
    SHOW SLAVE HOSTS
    SHOW SLAVE STATUS
    SHOW [GLOBAL | SESSION] STATUS [like_or_where]
    SHOW TABLE STATUS [FROM db_name] [like_or_where]
    SHOW [FULL] TABLES [FROM db_name] [like_or_where]
    SHOW TRIGGERS [FROM db_name] [like_or_where]
    SHOW [GLOBAL | SESSION] VARIABLES [like_or_where]
    SHOW WARNINGS [LIMIT [offset,] row_count]

    like_or_where:
        LIKE 'pattern'
      | WHERE expr
    ```

* 插入数据：

    ```mysql
    Syntax:
    INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
        [INTO] tbl_name [(col_name,...)]
        {VALUES | VALUE} ({expr | DEFAULT},...),(...),...
        [ ON DUPLICATE KEY UPDATE
          col_name=expr
            [, col_name=expr] ... ]

    Or:

    INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
        [INTO] tbl_name
        SET col_name={expr | DEFAULT}, ...
        [ ON DUPLICATE KEY UPDATE
          col_name=expr
            [, col_name=expr] ... ]

    Or:

    INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
        [INTO] tbl_name [(col_name,...)]
        SELECT ...
        [ ON DUPLICATE KEY UPDATE
          col_name=expr
            [, col_name=expr] ... ]
    ```

* 查询数据：

    ```mysql
    Syntax:
    SELECT
        [ALL | DISTINCT | DISTINCTROW ]
          [HIGH_PRIORITY]
          [STRAIGHT_JOIN]
          [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
          [SQL_CACHE | SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
        select_expr [, select_expr ...]
        [FROM table_references
        [WHERE where_condition]
        [GROUP BY {col_name | expr | position}
          [ASC | DESC], ... [WITH ROLLUP]]
        [HAVING where_condition]
        [ORDER BY {col_name | expr | position}
          [ASC | DESC], ...]
        [LIMIT {[offset,] row_count | row_count OFFSET offset}]
        [PROCEDURE procedure_name(argument_list)]
        [INTO OUTFILE 'file_name'
            [CHARACTER SET charset_name]
            export_options
          | INTO DUMPFILE 'file_name'
          | INTO var_name [, var_name]]
        [FOR UPDATE | LOCK IN SHARE MODE]]

    select * from user order by if(email is null ,0,1) ,email;
    Name: 'IF FUNCTION'
    Description:
    Syntax:
    IF(expr1,expr2,expr3)
    If expr1 is TRUE (expr1 <> 0 and expr1 <> NULL) then IF() returns
    expr2; otherwise it returns expr3. IF() returns a numeric or string
    value, depending on the context in which it is used.

    order by rand() limit 1 可以随机抽取一条结果；
    ```

    > 注：NULL不能进行比较运算，但可以使用 IS NULL或IS NOT NULL来进行判断；   
    having与where的相似之处是它们都是用来设定查询条件的，查询结果必须符合这些查询条件；   
    不同之处是count之类的统计函数的计算结果可以在having子句里出现；   

    _数据类型：_
    
    >  注：最好不要在同一个数据表里混合使用char和varchar类型。这是因为mysql会根据具体情况把数据列从一种类型转换为另一类型，规则如下：

      1. 在一个数据表里，如果每个数据列的长度都是固定的，那么每一个数据行的长度也是固定的；
      2. 只要数据表里有一个数据列的长度是可变的，那么各数据行的长度都将是可变的；
      3. 如果某个数据表里的数据行的长度是可变的，那么，为了节约存储空间，mysql会把这个数据表里的固定长度类型的数据列转换为相应的可变长度类型
      4. 只要表里有varchar、blog或text数据列，就不可能存在固定长度的char数据列了，默认会转换为varchar类型，但如果长度小于4个字符的char数据列不会被转换为varchar类型。
      5. mysql能够自动把两位数字的年份值转换为四位数字的年份值，规则如下：

        * 年份值00－69将被转换为2000－2069
        * 年份值70－99将被转换为1970－1999

* 清除一个数据表的全部内容将使有关序列从1开始重新编号，

    ```mysql
    delete from table_name;
    truncate table table_name;
    ```

    > 以上两条语句可以让auto_increment的编号值重新从1开始编号

* 在服务器查询哪些查询字符集可用：`show character set;`
* 想知道服务器的默认字符集是哪个：`show variables like 'character_set';`
* 查看数据库创建信息：`show create database test;`
* 对给定表，它的字符集用下面可以查出来：`show create table test_product;或show table status like 'test_product';`
* 匹配查询

    * %    能与任意个字符序列相匹配
    * _    只能与一个字符相匹配
    * .    用来匹配任何单个的字符
    * [..]    用来匹配在方括号内部出现的任何字符
    * -    用来给出一个字符范围，[1－5]
    * *    用来匹配“前一字符任意次数的连续重复出现”包括0次
    * ^part$    用来匹配子串出现在字符串的开头或末尾的情况
    * REGEXP 匹配串中任何的指定子串，如：`select * from pet where name regexp 'w';`

* 对数据库表添加权限：
    
    ```mysql
     GRANT ALL ON menagerie.* TO 'your_mysql_name'@'your_client_host';
    ```

* 从外部文件添加表记录：

    ```mysql
    mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet;
    ```

* 登录执行sql语句：

    ```mysql
    mysql -u root -p --execute="SELECT User, Host FROM mysql.user"
    ```

* 启动mysql服务：

    ```mysql
    mysqld_safe --port=port_num--defaults-file=file_name &
    ```

    > 指定启动时加载的配置文件--defaults-file=my.cnf，可以防止mysql优先读取/etc/my.cnf，

    > 因为在linux 系统中mysql配置文件的读取顺序为:   
    /etc/my.cnf   /etc/mysql/my.cnf    /usr/local/mysql/etc/my.cnf     ~/.my.cnf   
    或是：   
    ```mysql
    mysqladmin -uroot -p shutdown 
    sudo mysqld_safe &
    ```

* 修改root密码：`mysqladmin -u root -p password 新密码`
* 赋予任何主机访问数据的权限 ：

    ```mysql
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION 
    ##指定密码连接
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION
    ```

* having子句与where有相似之处但也有区别,都是设定条件的语句。

    > 在查询过程中聚合语句(sum,min,max,avg,count)要比having子句优先执行.而where子句在查询过程中执行优先级别优先于聚合语句(sum,min,max,avg,count),结论：

    1. WHERE 子句用来筛选 FROM 子句中指定的操作所产生的行。
    2. GROUP BY 子句用来分组 WHERE 子句的输出。
    3. HAVING 子句用来从分组的结果中筛选行。

* 通过sql脚本文件来创建数据库表：

    ```mysql
    mysql> source /home/zhangbingbing/test.sql
    ```

* MySQL不支持InnoDB的解决方法：

    ```mysql
    /var/lib/mysql目录下，删除ibdata1、ib_logfile1、 ib_logfile0，然后重启MySql让其重建以上文件: 
    mysqladmin -uroot -p shutdown 
    sudo mysqld_safe & 
    ```

* 查看表索引：

    ```mysql
    show index from muying_order;
    ```

* 配置查询日志及加载my.cnf配置：

    ```mysql
    查询日志在my.cnf中的[mysqld]添加log         = /home/zhangbingbing/local/mysql/log/mysqld_common.log
    同时修改my.cnf访问权限：chmod 644 my.cnf
    原因：my.cnf的读取权限进行了设置，不允许World-writable(字面意思是全世界都可读写)
    然后重新起动mysql服务，如果内存不足可以把innodb的配置设置的小点
    ```

* sql正则表达：

    ```mysql
    select from_unixtime(cast(unix_timestamp('{DATE}','yyyyMMdd') as bigint),'yyyy-MM-dd HH:mm:ss'),"母婴商品总点击pv", count(*)
    from udw.udw_event
    where event_day="{DATE}"
    and event_action="wise_wap_common_muying"
    and (wise_action_name="jumps" or wise_action_name="jump")
    -- and wise_statistics like "muying\_%\_ad_\_%"
    and wise_statistics REGEXP 'muying[_].*[_]ad[0-9][_].*'
    and event_isspider=0
    and event_isinternalip=0;
    ```

* mysql导出Sql语句：

    导出mysql数据库的user表

    ```mysql
    mysqldump -u root -p --opt mysql user
    ```

* 我们一般使用mysqldump和mysql：

    ```mysql
    mysqldump [options] db_name [tables]mysqldump [options] ---database DB1 [DB2 DB3...]mysqldump [options] --all--database
    ```

    如果没有指定任何表或使用了---database或--all--database选项，则转储整个数据库。 

* 备份一个数据库.

    ```mysql
    mysqldump -uroot -p123456 mysql > mysql_backup.sql
    ```

    > 这里备份了database mysql的结构和数据,生成的sql文件不会有创建database mysql的语句。
 
* 可以用一个命令备份mysql,test多个数据库：

    ```mysql
    mysqldump -u root -p123456 --database mysql test > my_databases.sql
    ```

    > 生成的sql文件有创建database mysql和test的语句
 
* 备份所有数据库：

    ```mysql
    mysqldump -u root -p123456  --all-databases > all_databases.sql
    ```

* 导出mysql这个数据库的结构

    ```mysql
    mysqldump -u root -p123456 -d --add-drop-table mysql > mysql_define.sql
    ```

* 导出一个数据所有数据并用gz压缩

    ```mysql
    mysqldump -u root -p123456 mysql | gzip > mysql.sql.gz
    ```
* 条件导出

    ```
    mysqldump -uroot -proot --databases db1 --tables a1 --where='id=1'  >/tmp/a1.sql
    mysqldump -uroot -proot --no-create-info --databases db1 --tables a1 --where="id='a'"  >/tmp/a1.sql
    ```

* 导出表结构不导出数据，--no-data

    ```
    mysqldump -uroot -proot --no-data --databases db1 >/tmp/db1.sql
    ```

* 可以这样将转储文件读回到服务器：

    ```mysql
    mysql db_name < backup-file.sqlmysql -e "source /path-to--backup/backup-file.sql" db_name
    或者从gz文件中还原
    gunzip -f < mysql.sql.gz | mysql -u root -p123456 test
    ```

* 解决mysqldump: Got error: 1044: Access denied for user

    ```mysql
    mysqldump -u dbuser -ppass db --skip-lock-tables > db.sql
    ```

    > 在导出时,加上--skip-lock-tables选项即可,但这个方法在数据量过大的话,会出现卡死,所以最好经常检查下sql备份,避免在要使用备份的时候杯具

* 解决is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'

   ```mysql
   mysql> flush hosts
   ```
* 查看并设置查询日志

   ```mysql
   mysql> SHOW VARIABLES LIKE 'general%';
   +------------------+---------------------------------------------------+
   | Variable_name    | Value                                             |
   +------------------+---------------------------------------------------+
   | general_log      | OFF                                               |
   | general_log_file | /home/work/.jumbo/var/lib/mysql/db-vp-novel-1.log |
   +------------------+---------------------------------------------------+
   2 rows in set (0.03 sec)

   mysql> set GLOBAL general_log='ON';
   Query OK, 0 rows affected (0.19 sec)

   mysql> SHOW VARIABLES LIKE 'general%';
   +------------------+---------------------------------------------------+
   | Variable_name    | Value                                             |
   +------------------+---------------------------------------------------+
   | general_log      | ON                                                |
   | general_log_file | /home/work/.jumbo/var/lib/mysql/db-vp-novel-1.log |
   +------------------+---------------------------------------------------+
   ```
   
* 查看表结构

```mysql
select COLUMN_NAME 列名, COLUMN_TYPE 字段类型, COLUMN_DEFAULT 默认值,IS_NULLABLE 是否可空, COLUMN_COMMENT 字段注释 from INFORMATION_SCHEMA.COLUMNS Where table_name = 'xxx_table' and table_schema ='xxx_db';
```
