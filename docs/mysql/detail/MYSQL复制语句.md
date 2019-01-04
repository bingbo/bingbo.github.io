
## 控制主服务器的语句

这里主要说下控制管理主服务器的一些语句语法，另外以下这些`SHOW`语句也用于主服务器上：

* SHOW BINARY LOGS
* SHOW BINLOG EVENTS
* SHOW MASTER STATUS
* SHOW SLAVE HOSTS

### PURGE BINARY LOGS语法

```sql
PURGE { BINARY | MASTER } LOGS
{ TO 'log_name' | BEFORE datetime_expr }
```

二进制日志是一系列的文件，这些文件包含了由mysql服务器对数据做的修改的一些信息。这些日志是由二进制日志文件及索引文件组成。

而`purge binary logs`语句将删除所有之前在日志索引文件中列出的指定日志文件名及日期的所有二进制日志文件。*BINARY*和*MASTER*是同义词，意思是一样的。删除日志文件同时也会删除索引文件中的记录列表，以便给定的日志将向前排列。

如果服务器没有用`--log-bin`选项开启二进制日志时，该删除日志语句是不起作用的。

例如：   
```sql
PURGE BINARY LOGS TO 'mysql-bin.010';
PURGE BINARY LOGS BEFORE '2008-04-02 22:46:26';
```

当从服务器正在复制数据时该语句会安全的运行，你不需要停止它们。如果你正在试图删除一个从服务器正在读的文件，则该语句不会做任何操作。为了安全的删除日志文件，可以按如下流程运行：   

1. 在从服务器上使用`show slave status`去检查正在读哪些日志文件
2. 通过`show binary logs`获取在主服务器上的一个二进制日志文件列表
3. 确定所有从服务器上最早的日志文件，这就是目标文件，如果从服务器都是最新的，则该文件就是清单上的最后一个日志文件
4. 针对所有你你删除的日志文件做备份
5. 删除所有的日志文件但不包括目标文件

你可以设置系统变量`expire_logs_days`使用日志在给定的时间后自动过期。`purge binary logs to`和`purge binary logs before`会以错误的形式失败，仅当在索引文件中的文件已经被系统以另外的方式移除的时候，这时候要手动编辑.index索引文件，删除不存在的文件重启即可。


### RESET MASTER语法

```sql
RESET MASTER
```

该语句删除在索引文件中的所有日志文件，并清空日志索引文件和新创建一个二进制日志文件。且该语句一般是在主服务器首次启动的时候使用。

> **重点关注**

> `RESET MASTER`的作用有两点不同于`PURGE BINARY LOGS`：

> 1. `RESET MASTER`会删除所有在索引文件中列表的二进制文件，仅留下一个空的以.000001为后缀的二进制文件，然而`PURGE BINARY LOGS`不会重置该序号。
> 2. `RESET MASTER`不会在任何复制从服务器运行的时候使用。然而`PURGE BINARY LOGS`可以在从服务器运行复制的进修安全运行。


`RESET MASTER`可能会很有用当你首次设置主服务器与从服务器时，你可以通过以下步骤去验证你的设置：

1. 启动主从服务器，开始复制
2. 执行一些测试查询在主服务器上
3. 检查查询是否被复制到了从服务器上
4. 当复制正在运行的时候，在从服务器`RESET SLAVE`命令后发布`STOP SLAVE`命令，然后检查是否不必要的数据已不存在在从服务器上。
5. 在主服务器上执行`RESET MASTER`命令清除所有的测试查询

在验证了设置和清除不必要的数据及测试查询日志之后，你可以启动从服务器进行复制了。

### SET sql_log_bin语法

```sql
SET sql_log_bin = {0|1}
```

`sql_log_bin`变量可以控制是否记录二进制日志，默认值是1，记录日志。如果要改变其值则需要超级权限来修改。


## 控制从服务器的语句

这里讨论下控制从服务器的语句，其中`SHOW SLAVE STATUS`和`SHOW RELAYLOG EVENTS`语句可以用于复制的从服务器。

### CHANGE MASTER TO语法

```sql
CHANGE MASTER TO option [, option] ...
option:
MASTER_BIND = 'interface_name'
| MASTER_HOST = 'host_name'
| MASTER_USER = 'user_name'
| MASTER_PASSWORD = 'password'
| MASTER_PORT = port_num
| MASTER_CONNECT_RETRY = interval
| MASTER_RETRY_COUNT = count
| MASTER_DELAY = interval
| MASTER_HEARTBEAT_PERIOD = interval
| MASTER_LOG_FILE = 'master_log_name'
| MASTER_LOG_POS = master_log_pos
| MASTER_AUTO_POSITION = {0|1}
| RELAY_LOG_FILE = 'relay_log_name'
| RELAY_LOG_POS = relay_log_pos
| MASTER_SSL = {0|1}
| MASTER_SSL_CA = 'ca_file_name'
| MASTER_SSL_CAPATH = 'ca_directory_name'
| MASTER_SSL_CERT = 'cert_file_name'
| MASTER_SSL_CRL = 'crl_file_name'
| MASTER_SSL_CRLPATH = 'crl_directory_name'
| MASTER_SSL_KEY = 'key_file_name'
| MASTER_SSL_CIPHER = 'cipher_list'
| MASTER_SSL_VERIFY_SERVER_CERT = {0|1}
| IGNORE_SERVER_IDS = (server_id_list)
server_id_list:
[server_id [, server_id] ... ]
```

`CHANGE MASTER TO`改变一些参数供从服务器用来连接主服务器，读取主服务器日志，读取从服务器中继日志等用。它也更新主服务器信息及中继日志信息到库中。在mysql 5.7.4及之后的版本，你可以使用`change master to `语句而不用再用`stop slave`来停止从服务器了。同时当某一个参数改变时，并不需要再指定所有的参数，只需修改某个参数就行，如只需要修改密码即可：

```sql
STOP SLAVE; -- if replication was running
CHANGE MASTER TO MASTER_PASSWORD='new3cret';
START SLAVE; -- if you want to restart replication
```

`master_host,master_user,master_password,master_port`等提供了从服务器如何连接主服务器的信息。如果你指定了`master_host`或`master_port`信息，则从服务器认为主服务器有所改变，这样老的信息被认为不在可用。如果设置`master_host,master_user`为空值会出现错误导致启动salve失败。

`MASTER_CONNECT_RETRY`指定了过多长时间再次重新连接，默认是60秒。

`master_retry_count`限制尝试重新连接的次数，其值可以在`show slave status`命令下看到，默认是24*3600=6400.

`master_delay`指定salve必须滞后于master多少秒，从master接收到的事件直到至少晚于master多少秒之后才会执行。默认是0.

`master_bind`用于有多个网络接口的复制salves上，且决定用哪个接口去连接master。

`master_heartbeat_period`设置了复制心跳之间间隔的秒数，每当主服务器的日志更新时，则下次心跳的等待周期会重置，间隔是一个十进制数，从0到4294967秒和毫秒级的分辨率，最小的值为0.001.心跳由主机发送仅当有关于二进制日志没有未发送的事件的时间比间隔时间长时。如果你记录master连接信息在表里，则`master_heartbeat_period`可以看作是mysql.slave_master_info表里heartbeat列表的值。如果设置interval为则心跳不可用，默认值等于_slave_net_timeout/2_.

`master_log_file`和`master_log_pos`指定了从服务器的I/O线程应该从主服务器日志的哪个地方开始读取。`relay_log_file`和`relay_log_pos`标明了从服务器的sql线程下次应该从中继日志的哪个地方开始读。如果指定了`master_log_file`或`master_log_pos`，则不能指定`relay_log_file`或`relay_log_pos`，如果指定了`master_log_file`或`master_log_pos`，则不能指定`master_auto_position=1`。如果`masger_log_file`和`master_log_pos`都没有指定的话，从服务器会使用`change_master_to`发布之前sql线程最后的位置坐标。如果在`change master to擛时候使用了`master_auto_position=1`，则从服务器会使用GTID复制协议去连接主服务器,当且仅当在从服务器的sql和IO线程停止时可以使用该选项。

`ignore_server_ids`是需要用逗号分隔的多个服务器的ID列表，来自这些相应服务器的事件都被忽略，日志轮换的异常及删除事件会仍记录在中继日志中。在循环复制中，原始服务器通常作为自己事件的终结者，以便他们应用不超过一次。这个选项在循环复制中当有一台服务器被删除时很有用，可以指定忽略该台机器。如果在`change master to`语句中没有指定`ignore_server_ids`选项，则所有现在的名单将被保存使用。所以要清除忽略的服务器列表，就必须使用该选项为一个空列表。

```sql
CHANGE MASTER TO IGNORE_SERVER_IDS = ();
```

`change master to`语句的行为依赖于从服务器的SQL线程和从服务器I/O线程的状态，这些线程是否在运行决定了哪些选择可以用于`change master to`语句，规则如下所示：

* 如果SQL线程处于停止状态，则你可以使用`relay_log_file`,`relay_log_pos`和`master_delay`选项的任意组合来运行`change master to`语句，即使I/O线程在运行。
* 如果I/O线程处于停止状态，则你可以使用任何选项来运行`change master to`语句，除了`relay_log_file`,`relay_log_pos`或`master_delay`，即使SQL线程在运行
* 当要在`change master to`语句中应用`master_auto_position=1`选项，则需要I/O线程和SQL线程都得停止。

你可以用`show slave status`语句来检查当前从服务器SQL线程和、I/O线程的状态。

下面的例子是改变从服务器使用的主服务器及建立从服务器开始读取的主服务器二进制日志位置坐标，这非常有用当时你设置slave去复制master时：

```sql
CHANGE MASTER TO
MASTER_HOST='master2.mycompany.com',
    MASTER_USER='replication',
    MASTER_PASSWORD='bigs3cret',
    MASTER_PORT=3306,
    MASTER_LOG_FILE='master2-bin.001',
    MASTER_LOG_POS=4,
    MASTER_CONNECT_RETRY=10;
```

下面的例子表明了一个极少使用的操作，当你因为某种原因想要重新执行中继日志时非常有用。要这样做，你需要使用`change master to`语句及开启SQL线程：

```sql
CHANGE MASTER TO
RELAY_LOG_FILE='slave-relay-bin.006',
    RELAY_LOG_POS=4025;
START SLAVE SQL_THREAD;
```

下表显示了一些字符串选项的值的最大长度：

| 参数选项 | 最大长度 |
| -------- | -------- |
|MASTER_HOST|60     |
|MASTER_USER| 16|
|MASTER_PASSWORD |32|
|MASTER_LOG_FILE |255|
|RELAY_LOG_FILE |255|
|MASTER_SSL_CA |255|
|MASTER_SSL_CAPATH |255|
|MASTER_SSL_CERT |255|
|MASTER_SSL_CRL |255|
|MASTER_SSL_CRLPATH |255|
|MASTER_SSL_KEY |255|
|MASTER_SSL_CIPHER |511|


### CHANGE REPLICATION FILTER语法

```sql
CHANGE REPLICATION FILTER filter[, filter][, ...]
filter:
REPLICATE_DO_DB = (db_list)
| REPLICATE_IGNORE_DB = (db_list)
| REPLICATE_DO_TABLE = (tbl_list)
| REPLICATE_IGNORE_TABLE = (tbl_list)
| REPLICATE_WILD_DO_TABLE = (wild_tbl_list)
| REPLICATE_WILD_IGNORE_TABLE = (wild_tbl_list)
| REPLICATE_REWRITE_DB = (db_pair_list)
db_list:
db_name[, db_name][, ...]
tbl_list:
db_name.table_name[, db_table_name][, ...]
wild_tbl_list:
'db_pattern.table_pattern'[, 'db_pattern.table_pattern'][, ...]
db_pair_list:
(db_pair)[, (db_pair)][, ...]
db_pair:
from_db, to_db
```

**change replication filter**设置了一个或多个复制过滤器规则，这个在启动slave服务时使用复制过滤选项如*－－replicate-do-db*或*--replicate-wild-ignore-table*等是一样的效果。该语句不需要重启服务器去生效，仅仅需要**stop slave sql_thread**语句停止SQL线程首先。

下面列表显示了**change replication filter**选项及相对应的*--replicate-\**等服务端选项：

* **replicate_do_db**:包含了要更新复制的数据库，与*--replicate-do-db*等效。
* **replicate_ignore_db**:要忽略的不更新复制的数据库，与*--replicate-ignore-db*等效。
* **replicate_ignore_table**:要忽略的不更新复制的数据表，与*--replicate-ignore-table*等效。
* **replicate_do_table**:要更新复制的数据表，与*--replicate-do-table*等效。
* **replicate_wild_do_table**:更新复制与指定模糊正则匹配的数据表，与*--replicate-wild-do-table*等效。
* **replicate_wild_ignore_table**:要忽略的不更新的与指定模糊正则匹配的数据表，与*--replicate-wild-ignore-table*等效。
* **replicate_rewrite_db**:指定要更新重写的数据库，与*--replicate-rewrite-db*等效。

其中**replicate_do_db**和**replicate_ignore_db**过虑器的效果依赖于是基于语句复制还是基于行的复制策略。也可以在单个的**change replication filter**语句中运用多个过滤器规则，如果指定相同的规则多次，则最后一次是有用的：

```sql
CHANGE REPLICATION FILTER
REPLICATE_DO_DB = (d1), REPLICATE_IGNORE_DB = (d2);
CHANGE REPLICATION FILTER
REPLICATE_DO_DB = (db3,db4);
```

> 这里的行为不同与*--replicate-\**过虑选项，指定多次相同的服务端选项，则会造成创建多个过虑器规则。

**replication_wild_table**和**replication_wild_ignore_table**选项的值是字符串表达式，可能会包含模糊操作符，所以必须用引号括起来：

```sql
CHANGE REPLICATION FILTER
REPLICATE_WILD_DO_TABLE = ('db1.old%');
CHANGE REPLICATION FILTER
REPLICATE_WILD_IGNORE_TABLE = ('db1.new%', 'db2.new*');
```

**replication_rewrite_db**值的表现形式是数据库对，每一数据库对都用括号括起来，下面的例子创建了两个重写规则，重写dbA到dbB,dbC到dbD：

```sql
CHANGE REPLICATION FILTER
REPLICATE_REWRITE_DB = ((dbA, dbB), (dbC, dbD));
```

删除相应的过滤规则，则置相应的选项为空即可：

```sql
CHANGE REPLICATION FILTER
REPLICATE_DO_DB = (), REPLICATE_IGNORE_DB = ();
```



### MASTER_POS_WAIT()语法

```sql
SELECT MASTER_POS_WAIT('master_log_file', master_log_pos [, timeout])
```

这其实是一个函数，而非语句，它常常用于确保从服务器读取并执行相应的事件到在主服务器日志文件中指定的位置。

### RESET SLAVE语法

```sql
RESET SLAVE [ALL]
```

**reset slave**使slave忘记主服务器日志文件需要复制的位置信息。意味着清除从头开始，它会清除 主服务器及中继日志信息库，删除所有的中继日志文件，并开始一个新的中继日志文件，使用**reset slave**语句必须要停止从服务器的复制线程，如果必要的话就用**stop slave**语句。

> 所有的中继日志都会被删除，即使它们没有被从服务器的SQL线程执行完。

所有的连接信息参数都可以通过**reset slave all**被重置，也可以在重启从服务器后妊**reset slave**来实现。当从服务器的SQL线程停止时，会被放在复制临时表中，如果这时执行**reset slave**命令则会删除这些在从服务器上的临时表。

### SET GLOBAL sql_slave_skip_counter语法

```sql
SET GLOBAL sql_slave_skip_counter = N
```

该语句设置跳过来自主服务器的接下来的N个事件，这对从被一个语句造成的复制停止中恢复是很有用的。该语句仅当从服务器线程没有运行时使用是合法的，否则会产生一个错误。当使用该语句时，对于理解二进制日志其实就是被认为是一组序列组成的事件组是很重要的，每个事件组又是由一系列的事件组成。

### START SLAVE语法

```sql
START SLAVE [thread_types] [until_option] [connection_options]
thread_types:
[thread_type [, thread_type] ... ]
thread_type:
IO_THREAD | SQL_THREAD
until_option:
UNTIL { {SQL_BEFORE_GTIDS | SQL_AFTER_GTIDS} = gtid_set
| MASTER_LOG_FILE = 'log_name', MASTER_LOG_POS = log_pos
| RELAY_LOG_FILE = 'log_name', RELAY_LOG_POS = log_pos
| SQL_AFTER_MTS_GAPS }
connection_options:
[USER='user_name'] [PASSWORD='user_pass'] [DEFAULT_AUTH='plugin_name'] [PLUGIN_DIR='plugin_dir']
gtid_set:
uuid_set [, uuid_set] ...
| ''
uuid_set:
uuid:interval[:interval]...
uuid:
hhhhhhhh-hhhh-hhhh-hhhh-hhhhhhhhhhhh
h:
[0-9,A-F]
interval:
n[-n]
(n >= 1)
```

**start slave**语句如果不带线程类型，则会启动从服务器的两种线程。I/O线程读取来自主服务器的事件，并存储在中继日志中。SQL线程读取来自中继日志的事件并执行他们。同时执行**start slave**需要超级权限。

如果启动slave服务成功，则不会返回错误，否则它会先启动，过会就会停止。且该语句不会给你任何提示，你必须检查错误日志或是用**show slave status**检查状态。也可以启动从服务带上**user**,**password**,**default_auth**和**plugin_dir**等选项：

* **user**:用户名
* **password**:密码
* **default_auth**:插件的名称，默认是mysql 内置的验证
* **plugin_dir**:插件位置

> 你可以用**show processlist**语句查看整个从服务器的启动运行状态，包含用户名或密码，这里也包含运行**change master to**语句设置的有关主服务器的信息，如*master_user*或*master_password*。

**start slave**语句会在I/O线程和SQL线程都启动后给用户发送认可消息，但是I/O线程可能会没有连接上，这时通过**show slave status**可以查看到**slave_sql_running=yes**，但并不会保证**slave_io_running=yes**，仅当只有I/O线程运行时。

你可以指定**IO_THREAD**和**SQL_THREAD**选项在该语句后面来指定启动相应的线程。

**until**语句可以用来指定slave应该启动运行直到SQL线程到达master日志某个给定的点，这个点可以通过**master_log_pos**和**master_log_file**选项指定，或者是用**relay_log_pos**和**relay_log_file**选项指定的中继日志中的某个点。当SQL线程到达某个指定的点，则停止。你不能在**IO_THREAD**选项中使用**until**语句。

**until**语句不支持多线程的从服务器，除非使用**sql_after_mts_gaps**选项。要使用**until**语句，则你必须指定以下选项的其中一项：

* 日志名和日志位置
* **sql_before_gtids**或**sql_after_gtids**
* **sql_after_mts_gaps**



### STOP SLAVE语法

```sql
STOP SLAVE [thread_types]
thread_types:
[thread_type [, thread_type] ... ]
thread_type: IO_THREAD | SQL_THREAD
```

停止slave线程，该操作需要超级权限。建议在停止从服务器时要先执行**stop slave**。如果你正在复制日志，且使用的是基于行的日志格式，则在关掉从服务器之前你应该执行**stop slave**或**stop slave sql_thread**。
