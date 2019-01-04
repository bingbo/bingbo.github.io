## mysql客户端显示中文乱码问题

1. 查看表字符编码

```mysql
mysql> show create table user\G
*************************** 1. row ***************************
Table: user
Create Table: CREATE TABLE `user` (
         `id` int(11) NOT NULL AUTO_INCREMENT,
           `name` varchar(10) DEFAULT NULL,
             `password` varchar(20) DEFAULT NULL,
               `email` varchar(100) DEFAULT NULL,
                 PRIMARY KEY (`id`)
       ) ENGINE=InnoDB AUTO_INCREMENT=37 DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

```

2. 创建表的时候指定字符编码

```mysql
create table user(name varchar(11)) default charset=utf8; 
```

3. 查看服务器系统编码

```bash
$locale
LANG="zh_CN.UTF-8"
LC_COLLATE="zh_CN.UTF-8"
LC_CTYPE="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
LC_MONETARY="zh_CN.UTF-8"
LC_NUMERIC="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"
LC_ALL="zh_CN.UTF-8"
```

4. 查看数据库编码

```mysql
mysql> show variables like '%char%';
+--------------------------+--------------------------------------------------------+
| Variable_name            | Value                                                  |
+--------------------------+--------------------------------------------------------+
| character_set_client     | utf8                                                   |
| character_set_connection | utf8                                                   |
| character_set_database   | utf8                                                   |
| character_set_filesystem | binary                                                 |
| character_set_results    | utf8                                                   |
| character_set_server     | utf8                                                   |
| character_set_system     | utf8                                                   |
| character_sets_dir       | /usr/local/Cellar/mysql/5.7.18_1/share/mysql/charsets/ |
+--------------------------+--------------------------------------------------------+
8 rows in set (0.02 sec)
```

5. 在session范围内设置字符变量值

```bash
set character_set_server=utf8;
set character_set_database=utf8;
set character_set_results=utf8;
```

6. 在全局范围内设置字符值

```bash
set global character_set_database=utf8;
set global character_set_server=utf8;
```

7. 可以修改配置文件my.cnf设置字符

```bash
[mysqld]
character-set-server=utf8 
[client]
default-character-set=utf8 
[mysql]
default-character-set=utf8
```

> show命令的应用


```bash
mysql> help show;
Name: 'SHOW'
Description:
SHOW has many forms that provide information about databases, tables,
     columns, or status information about the server. This section describes
     those following:

     SHOW {BINARY | MASTER} LOGS
     SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count]
     SHOW CHARACTER SET [like_or_where]
     SHOW COLLATION [like_or_where]
     SHOW [FULL] COLUMNS FROM tbl_name [FROM db_name] [like_or_where]
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
     SHOW MASTER STATUS
     SHOW OPEN TABLES [FROM db_name] [like_or_where]
     SHOW PLUGINS
     SHOW PROCEDURE CODE proc_name
     SHOW PROCEDURE STATUS [like_or_where]
     SHOW PRIVILEGES
     SHOW [FULL] PROCESSLIST
     SHOW PROFILE [types] [FOR QUERY n] [OFFSET n] [LIMIT n]
     SHOW PROFILES
     SHOW RELAYLOG EVENTS [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count]
     SHOW SLAVE HOSTS
     SHOW SLAVE STATUS [NONBLOCKING]
     SHOW [GLOBAL | SESSION] STATUS [like_or_where]
     SHOW TABLE STATUS [FROM db_name] [like_or_where]
     SHOW [FULL] TABLES [FROM db_name] [like_or_where]
     SHOW TRIGGERS [FROM db_name] [like_or_where]
     SHOW [GLOBAL | SESSION] VARIABLES [like_or_where]
     SHOW WARNINGS [LIMIT [offset,] row_count]

     like_or_where:
         LIKE 'pattern'
           | WHERE expr

           If the syntax for a given SHOW statement includes a LIKE 'pattern'
           part, 'pattern' is a string that can contain the SQL % and _ wildcard
           characters. The pattern is useful for restricting statement output to
           matching values.

           Several SHOW statements also accept a WHERE clause that provides more
           flexibility in specifying which rows to display. See
           http://dev.mysql.com/doc/refman/5.7/en/extended-show.html.

URL: http://dev.mysql.com/doc/refman/5.7/en/show.html
```

> set命令的应用

```bash
mysql> help set;
Name: 'SET'
Description:
Syntax:
SET variable_assignment [, variable_assignment] ...

variable_assignment:
      user_var_name = expr
      | param_name = expr
      | local_var_name = expr
      | [GLOBAL | SESSION]
          system_var_name = expr
      | [@@global. | @@session. | @@]
              system_var_name = expr

              SET syntax for variable assignment enables you to assign values to
              different types of variables that affect the operation of the server or
              clients:

              o System variables. See
                http://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html.
                  System variables also can be set at server startup, as described in
                    http://dev.mysql.com/doc/refman/5.7/en/using-system-variables.html.
                      (To display system variable names and values, use the SHOW VARIABLES
                         statement; see [HELP SHOW VARIABLES].)

                      o User-defined variables. See
                        http://dev.mysql.com/doc/refman/5.7/en/user-variables.html.

                        o Stored procedure and function parameters, and stored program local
                          variables. See
                            http://dev.mysql.com/doc/refman/5.7/en/stored-program-variables.html.

URL: http://dev.mysql.com/doc/refman/5.7/en/set-variable.html
```
