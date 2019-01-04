复制可以把数据从一个mysql数据库服务器(master)复制到另外的其他mysql数据库服务器(slave)，默认情况下该过程是异步的；从服务器不需要持久的连接并接收来自主服务器的更新。复制依赖于配置，你可以复制所有的数据库，已选择的数据库或选择复制一个数据库中的一些表。

mysql复制的优点：

* 向外扩展解决方案-分布负载到多个从服务器之间，从而提高性能。在这种情况下，所有的写操作都在主服务器上，所有的读操作在一个或多个从服务器上，这样可以提高写的性能，可以动态的增加读的速度通过增加从服务器的数量。
* 数据安全-因为数据会复制到从服务器上，同时从服务器可以随时停止复制，这样在从服务器上进行备份服务而不会影响相应的主服务器的数据。
* 分析-实时的数据可以在主服务器上创建，而在从服务器上进行数据分析而不影响主服务器的性能。
* 长距离数据分布-你可以通过复制为远程的站点创建一个本地版本的数据，而不需要持久的访问主从服务器。

## 同步复制配置

### 如何设置复制

这里有多种不同的方法来设置主从复制，使用哪种方法依赖于你如何设置复制，且在你的主服务器数据库中是否已有数据。这里有一般针对复制设置来说的一些通用的任务：

* 在主服务器上，你必须打开二进制日志且配置一个唯一的服务器ID，这可能需要重启服务器
* 在每一台你想要连接主服务器的从服务器上，你必须配置一个唯一的服务器ID，这也可能需要重启服务器。
* 在者，有可能需要为你的从服务器创建一个用户，为了当读取主服务器二进制日志复制时做验证使用。
* 在创建数据快照或开始复制过程时，你需要记录主服务器上二进制日志的位置，当配置从服务器时你需要这些信息来告诉从服务器从哪里开始执行事件。
* 如果主服务器上已经存在一些数据且想同步到从服务器，你需要创建一个数据快照复制数据到从服务器上。而你使用的存储引擎会影响你如何创建数据快照。当你使用**MyISAM**时，你必须停止处理语句在主服务器上获取读锁，然后获取当前二进制日志的位置坐标并转存数据在允许主服务器继续执行语句之前。如果你不停止处理语句，则你转存(dump)的数据与主服务器的状态信息会不匹配，导致数据不一致；如果你使用的是**InnoDB**引擎，则你不需要获取读锁，一个足够长时间发送数据的传输就足够了。
* 配置从服务器，设置到主服务器的连接，如主机名，登录凭证，二进制日志的文件名及位置。

配置了基本选项之后，就可以选择你的场景了：

* 为一个新装的没有任何数据的主从服务器设置复制
* 用已存在的mysql服务器上的数据为一个新的主服务器进行设置复制
* 添加从服务器到一个已存在的复制环境中

> **设置复制主服务器的配置**

在主服务器上必须开启二进制日志，因为二进制日志是从主服务器到从服务器复制更新变化的基础，如果没有用**log-bin**选项打开二进制日志，则复制是不可以会进行的。

在一个复制组中的每一个服务器必须配置一个唯一的服务器ID，这个ID用于表明组内的单个服务器，并且其值必须在1和(2的32次方)–1之间。

为了配置二进制日志及服务器ID选项，需要关闭mysql服务器并编辑**my.cnf**或**my.ini**文件，在其**[mysqld]**块中添加**log-bin**和**server-id**选项，如：

```sql
[mysqld]
log-bin=mysql-bin  #二进制日志文件名前缀
server-id=1
```

完成这些配置后，重启服务器即可。

> _下面这些会影响你的过程：_     

* 如果忽略了server-id或是server-id＝0则master拒绝任何slave的连接
* 为了保证最大的持久性及一致性时，会使用InnoDB的事务特性，那么你应该使用*innodb_flush_log_at_trx_commit=1*和*sync_binlog=1*在master的my.cnf配置文件中
* 确保skip-networking选项没有打开在master上，如果网络不可用，slave就不能与master通信，复制就会失败


> **设置复制从服务器的配置**

如果从服务器ID还没有设置，或是当前的值与主服务器的一些ID冲突，则你需要关闭从服务器编辑**[mysqld]**块去指定一个唯一的服务器ID。如：

```sql
[mysqld]
server-id=2
```

完成配置后，重启服务器即可。如果你设置了多个从服务器，则每个服务器必须有一个唯一的*server-id*，该ID不同于主服务器及所有的其他从服务器。如果你 省略了服务器ID，则从服务器拒绝连接主服务器。

你不必在从服务器上为复制开启二进制日志，然而如果你在从服务器上开启了二进制日志，则你可以使用从服务器上的二进制日志进行数据备份和故障恢复，还可以使用从服务器做为一个更的复杂的复制结构的一部分。


> **为复制创建一个用户**

每一个从服务器连接主服务器时需要用户名和密码，所以在主服务器上必须有一个账号供从服务器用来连接。你可以使用**replication slave**权限为每一个账号授权，也可以为每一个从服务器选择创建一个不同的账号，或使用一样的账号连接主服务器。

使用**create user**语句创建一个新的账号，并用**grant**语句为该账号授权，如：

```sql
mysql> CREATE USER 'repl'@'%.mydomain.com' IDENTIFIED BY 'slavepass';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%.mydomain.com';
```

> **获取复制主服务器二进制日志的位置坐标**

为了配置从服务器能在正确的位置进行复制，你需要知道主服务器当前的二进制日志坐标位置。

> *警告*    
此过程使用**flush tables with read lock**以阻止对InnoDB表的commit操作。

为了获取主服务器的二进制日志坐标位置，按以下步骤运行：

1. 开启一个会话，刷新所有的表及阻止写操作，用`mysql> FLUSH TABLES WITH READ LOCK;`，注意：执行语句后保持该会话连接，如果你退出该连接，则锁被释放。
2. 在另一个主服器会话上，使用**show master status**显示当前的二进制日志文件及位置：

    ```sql
    mysql > SHOW MASTER STATUS;
    +------------------+----------+--------------+------------------+
    | File | Position | Binlog_Do_DB | Binlog_Ignore_DB |
    +------------------+----------+--------------+------------------+
    | mysql-bin.000003 | 73 | test | manual,mysql |
    +------------------+----------+--------------+------------------+
    ```

例子中的文件是*mysql-bin.000003*且位置是73,记录这些值，在配置从服务器时需要，它们表示了从服务器开始复制更新的位置坐标。如果主服务器先前运行时没有开启二进制日志，则通过**show master status**或**msyqldump --master-data**语句显示的日志文件及位置将是空的，这种情况下配置从服务器时其文件及位置应该为''或4。



> **选择数据快照的方法**

如果主服务器数据库包含已存在的数据，则必须把数据复制到每一个从服务器上。这里有不同的方法去从主服务器数据库dump数据，下面是可能的选项。

* 使用**mysqldump**工具创建你要复制的所有数据库的转存备份，这是推荐方法，特别是你使用的是InnoDB引擎。
* 如果你的数据库存储在二进制文件中，则你可以复制原始的数据文件到从服务器，这可能比使用**mysqldump**并导入数据到每一个从服务器更有效，因为它跳过了当insert语句重新执行时更新索引的开销。InnoDB引擎是不推荐使用的。

**使用*mysqldump*创建一个数据快照**

下面的例子转存所有的数据为到一个dbdump.db的文件中，其中包含了*--master-data*选项，这样会自动带上从服务器开始复制过程所需要的*change master to*语句：

```sql
shell> mysqldump --all-databases --master-data > dbdump.db
```

如果你没有使用*--master-data*选项，则你需要在另一个独立的会话中手动锁定所有的表。如果需要去掉相应的数据库，则在使用mysqldump工具时不要使用*--all-databases*,可以使用以下选项：

* 不包含所有的表，则使用*--ignore-table*选项
* 使用*--databases*选项指定相应的数据库

为了导入数据，要么复制dump的文件到从服务器，要么远程连接从服务器并访问主服务器上的文件。

**使用原生数据文件创建数据快照**

为了创建原生数据快照，针对MyISAM表来说，当你的数据文件存在于单独的一个文件系统中，你可以使用标准的复制工具如*cp*或*copy*，远程复制工具如*scp*或*rsync*，一个打包工具如*zip*或*tar*。如果你只复制特定的数据库，仅复制相应的文件即可。而对于InnoDB表来说，所有的数据库数据表都存储在**system tablespace**文件中，除非你开启了*innodb_file_per_table*选项。

下面这些文件是不需要复制的：

* 与*mysql*数据库相关的文件
* 主服务器信息库文件
* 主服务器器的二进制文件
* 任何中继日志文件

这依赖于你使用的是否是InnoDB表，如果是InnoDB表，为获得一致的原生数据快照，需要关闭主服务器并按如下流程：

1. 获取读锁并获得主服务器的状态
2. 在一个单独的会话中，关闭主服务器：`shell> mysqladmin shutdown`
3. 复制mysql数据文件。

    ```shell
    shell> tar cf /tmp/db.tar ./data
    shell> zip -r /tmp/db.zip ./data
    shell> tar cf /tmp/db.tar ./data
    shell> zip -r /tmp/db.zip ./data
    ```

4. 重启主服务器。

如果你没有使用InnoDB表，则按如下流程：

1. 获取读锁并获得主服务器的状态
2. 复制mysql数据文件

    ```shell
    shell> tar cf /tmp/db.tar ./data
    shell> zip -r /tmp/db.zip ./data
    shell> rsync --recursive ./data /tmp/dbdata
    ```

3. 释放锁，`mysql> unlock tables;`

> **设置所有复制的从服务器**

确保你已经做了如下流程：

* 已经配置了mysql的主服务器的一些必要属性
* 获取了主服务器的状态信息
* 在主服务器上释放读锁：`mysql> unlock tables;`
* 在从服务器上，编辑配置mysql配置


**针对新的主从服务器进行复制配置**

1. 启动从服务器
2. 执行*change master to*语句设置主服务器信息

在每一个从服务器上执行这些操作。如果你正在设置一个新的服务器但已经有来自不同服务器的dump文件，而你想加载这些到你的复制配置中时，这种方法也是有用的。通过加载数据到新的主服务器上，这些数据会自动复制到从服务器上。如果你正在设置一个新的复制环境，使用来自不周服务器的数据创建一个新的主服务器，则在新主服务器上运行dump文件，数据库会自动传播到从服务器上：`shell> mysql -h master < fulldb.dump`

**使用已有的数据进行复制设置**

如果你使用的是*mysqldump*：

1. 启动从服务器，使用*--skip-slave-start*选项以复制不开始
2. 导入dump文件：`shell> mysql <fulldb.dump`

如果你使用的是原生数据文件创建的快照：

1. 抽取数据文件到你的从服务器的数据目录中，如：`shell> tar xvf dbdump.tar`，可能需要设置下权限以便服务器可以访问这个文件。
2. 启动从服务器，使用*--skip-slave-start*选项以复制不开始
3. 配置从服务器从主服务器开始复制的位置坐标
4. 启动从服务器线程。`mysql> start slave;`

从服务器使用存储在主服务器库中的信息来追踪主服务器的日志进行了多少。该库可以是文件也可以是表的形式，这决定于*--master-info-repository*的值，当从服务器以*--master-info-repository=File*的形式运行时，你可以在它的数据目录中找到*master.info*和*relay-log.info*两个文件。如果是*--master-info-repository=Table*，则信息存储在*master_slave_info*表中。一般不要直接修改这些文件及表信息，可以以*change master to*语句的形式修改。


> **引入额外的从服务器到现在有的复制环境**

为了在已有的复制配置中添加另一从服务器，你可以不用停止主服务器，直接通过做一个已存在的从服务器的copy来设置新的从服务器，除非你用新的服务器ID配置了新的从服务器。

为了复制已存在的从服务器：

1. 关闭已存在的从服务器：`shell> mysqladmin shutdow`
2. 复制数据目录从已有从服务器到新从服务器，你可以通过打包复制。
3. 复制主服务器信息和中继日志信息库从已有slave到新slave
4. 开启已有slave
5. 在新的slave上编辑配置并给新的slave一个唯一的服务器ID
6. 启动新的slave，该从服务器使用主服务器信息库开始复制过程


> **在从服务器上设置主服务器信息**

为了设置从服务器与主服务器通信进行复制，你必须告诉从服务器一些必要的连接信息，则执行以下语句在slave上：

```sql
mysql> CHANGE MASTER TO
-> MASTER_HOST='master_host_name',
    -> MASTER_USER='replication_user_name',
    -> MASTER_PASSWORD='replication_password',
    -> MASTER_LOG_FILE='recorded_log_file_name',
    -> MASTER_LOG_POS=recorded_log_position;
```

复制不能用unix套接字，你必须要用TCP/IP连接到主服务器。




### 主从复制方式

复制可以工作是因为事件被写入到从master读取的二进制日志文件中，并在slave上处理。不同的复制方式会使用相应的二进制日志格式：

* 当使用基于语句的二进制日志时，master写SQL语句到二进制日志中，从master到slave的复制通过在slave上执行SQL语句工作。这就是所谓的基于语句的复制。
* 当使用基于行的的日志时，master写事件到二进制日志里表明个别表的行是如何变化的。从master到slave上的复制是通过复制表行变化的事件到slave上实现的，这就是所谓的基于行的复制。
* 你还可以配置mysql使用基于语句和行的混合日志格式，这取决于哪种方式最适合于记录的变化。默认情况下使用基于语句的复制方式，在特定的语句及相应的存储引擎下，日志自动会切换到基于行的复制方式下运行。

你可以使用超级权限通过*binlog_format*选项进行设置日志方式。

> **基于语句和行的复制的优缺点**

**基于语句复制的优点**

* 较少的数据写入日志文件
* 日志文件包含所有的进行变化的sql语句，所以常用来审核数据库

**基于语句复制的缺点**

* 一些语句针对基于语句的复制来说是不安全的。并不是所有的修改数据的语句都可以复制，一些不确定性的发生是很难用基于语句的复制方式来实现的，如下语句：

    * 全速于UDF或存储程序的语句是不确定的，因为它们返回的值依赖于提供的参数
    * 使用了*limit*而没有*order by*的**delete**和**update**语句是不确定的
    * 确定性的UDF必须应用于从服务器上
    * 使用下列函数的语句不能使用基于语句的复制方式来复制：

        * LOAD_FILE()
        * UUID(), UUID_SHORT()
        * USER()
        * FOUND_ROWS()
        * SYSDATE()
        * GET_LOCK()
        * IS_FREE_LOCK()
        * IS_USED_LOCK()
        * MASTER_POS_WAIT()
        * RAND()
        * RELEASE_LOCK()
        * SLEEP()
        * VERSION()

    * **insert...select..**需要比基于行的复制的更大数据量的行级锁
    * **update**语句需要全表扫描，需要比基于行的复制锁定更多的行
    * 针对InnoDB引擎，一个使用**auto_increment**的**insert**语句会阻碍其他的**insert**语句
    * 对于复杂的语句来说，需要在更新或插入之前必须在从服务器上进行评估与执行过。而用基于行的复制，从服务器只修改影响的行，而不执行所有的语句。
    * 如果在评估中有错误发生，特别是当执行复制语句时，基于语句的复制可能会慢慢的增加误差。
    * 表的定义必须在主从服务器上一致。

**基于行复制的优点**

* 所有的改变都可以被复制，这是复制最安全的方式。
* 该技术与大多数其他数据库管理系统是一样的。
* 在master上需要更少的行锁，因此能达到更高的并发性，针对以下语句：
    
    * insert ... select
    * 带auto_increment的insert语句
    * 带where语句的update或delete语句，它们不会使用keys或是不会改变太多的行

* 在从服务器上的**insert,update,delete**语句会需要更少的行锁

**基于行复制的缺点**

* 基于行的复制会生成大量的日志数据。对于复制一个数据操作语句，如update或delete，基于语句的复制仅写语句到二进制日志中，而相对来说，基于行的复制会写每个改变的行到日志中，如果语句改了多行，则会写大量的数据到日志中，即使是回滚语句。这也意味着从备份中应用和恢复数据会需要大量的时间。另外日志文件会被锁定很长一段时间来写数据，会造成不一致的问题。
* 可能会比基于语句的复制花费更多的时间去复制**BLOG**类型的数据。
* 你不能通过检查日志来查看哪些语句被执行了，也不能在从服务器看到哪些语句是从主服务器收到的且执行过的。
* 针对用 MyISAM引擎的表来说，在从服务器上执行insert语句来复制日志数据时会需要更强的锁机制，这意味着使用基于行的复制时，不支持对MyISAM表进行并发插入。

> **基于行的记录和复制的使用**

mysql使用基于语句的日志，基于行的或混合格式的日志，二进制日志的类型影响了日志的大小及效率，所以选择使用哪种类型要依赖于你的应用及环境。

* 会产生大量的日志，同时需要花大量的时间去复制同步数据，从而会导致延时。
* 可以用**mysqlbinlog**读取日志 
* 如果**slave_exec_mode**指定为IDEMPOTENT(幂等)，则处理错误可能不会被发现，同步复制的表会产生很大的差异，不同步等。
* 缺少日志校验。基于行的日志记录不会进行校验，所以在处理二进制日志中可能不会标明错误信息。
* 不支持基于服务器ID的过滤器，在5.7版本可以使用**ignore_server_ids**选项在**change master to**语句中来过滤server id。
* 建议避免使用数据库级别的复制，用表级别的复制选项代替，如*--replicate-do-table*和*--replicate-ignore-table*。
* 当使用基于行的记录时，如果在从服务器线程在更新非事务表时服务器停止了，这会导致从服务器复制了一个不一致的表，所以建议使用事务性存储引擎创建要使用行的复制的表。

> **二进制日志安全和不安全的确定**

这里安全不安全主要是看能不能正确的进行同步复制数据。

安全与不安全语句的处理：

* 当使用基于行的日志记录时，针对安全与否来说，处理是一样的。
* 当使用混合方式的日志记录时，标记为不安全的语句使用的是基于行的格式记录的;认为是安全的语句是使用基于语句的格式记录。
* 当使用基于语句的日志记录时，对于不安全的语句会产生一条警告信息，安全的则正常执行。

下面格式的语句被认为是不安全的：

* 包含系统函数的语句且在从服务器上会返回不同的值，这些函数有FOUND_ROWS(), GET_LOCK(), IS_FREE_LOCK(), IS_USED_LOCK(),LOAD_FILE(), MASTER_POS_WAIT(), PASSWORD(), RAND(), RELEASE_LOCK(), ROW_COUNT(),SESSION_USER(), SLEEP(), SYSDATE(), SYSTEM_USER(), USER(), UUID(), and UUID_SHORT()
* 系统变量
* 全文插件，该插件在不同的服务器上有不同的行为，而依赖于它的语句也会有不同的结果
* 触发器和存储过程更新有auto_increment列的表时
* 在有混合主键或唯一索引的表上执行insert...on duplicate key update语句
* 带有limit的update语句
* LOAD DATA INFILE语句

### 复制和二进制日志功能选项和变量

下面包含了用来在复制中控制日志的*mysqld*选项及服务器变量.

* server-id

    > 在master和slave上，为每个服务器分配的唯一的ID，从0-4294967295，默认值为0。


:corn: **复制和二进制日志功能选项和变量参考**

* log-slave-update

    > 这个选项告诉slave记录由SQL线程执行的更新到自己的二进制日志中

* log_slave_updates

    > 告诉slave是否应该记录有SQL线程执行的更新到自己的二进制日志中

* master-info-file

    > 记住master及I/O复制线程在master日志中的位置的文件的位置与名字

* master-info-repository｜master_info_repository

    > 是否记录master的状态信息和I/O线程位置到文件或是表中

* master-retry-count

    > slave连接master的重试次数，在放弃之前

* relay-log

    > 中继日志的位置及基本名字

* relay-log-index

    > 中继日志索引文件的位置名字

* relay-log-info-file

    > 一个文件的位置及名字，该文件会记住SQL线程在路线日志的什么位置


* relay_log_info_file

    > 记录中继日志信息的文件的名字

* relay-log-info-repository|relay_log_info_repository

    > 是否写入在中继日志复制的SQL线程位置到文件或一个表

* relay_log_index

    > 中继日志索引文件名字

* relay_log_purge

    > 是否要删除中继日志

* relay-log-recovery

    > 启用在启动是从master自动恢复中继日志文件

* relay_log_recovery

    > 是否启用中继日志文件的自动恢复机制，对防撞的slave必须启用

* relay_log_space_limit

    > 中继日志可以使用的最大空间

* replicate-do-db|--replicate-do-db=db_name

    > 告诉从服务器的SQL线程限制复制指定的数据库

* replicate-do-table

    > 告诉从服务器的SQL线程限制复制指定的表

* replicate-ingore-db

    > 告诉从服务器的SQL线程不复制指定的数据库

* replicate-ignore-table

    > 告诉从服务器的SQL线程不复制指定的表

* replicate-rewrite-db

    > 用不同的名字代替更新一个数据库

* replicate-same-server-id
    
    > 在复制中，如果为1,则不要跳过有我们服务器ID的事件

* replicate-wild-do-table

    > 告诉slave线程只复制匹配正则表达式的表

* replicate-wild-ignore-table

    > 告诉slave线程不要复制匹配正则表达式的表

* report-host

    > 在slave注册过程中，向master报告slave的主机名或IP

* report-password

    > 从服务器的密码应报告给主服务器，不同的复制账号有不同的密码

* report-port

    > 报告用于连接从服务器的端口给主服务器，在从服务器注册过程中

* report-user

    > 报告从服务器复制账号的用户名到主服务器

* rpl_semi_sync_master_enabled

    > 是否开启主服务器的半同步复制

* rpl_semi_sync_master_timeout

    > 毫秒数等待从服务器确认
* rpl_semi_sync_master_trace_level

    > 主服务器半同步复制调试跟踪级别

* rpl_semi_sync_master_wait_for_slave_count

    > 在每个事务处理之前主服务器必须收到从服务器确认的次数

* rpl_semi_sync_master_wait_no_slave

    > 在没有slave的情况下，主服务器是否启用等待超时

* rpl_semi_sync_master_wait_point

    > 从服务器事件收到确认的等待点

* rpl_semi_sync_slave_enabled

    > 是否在从服务器上开启半复制

* rpl_semi_sync_slave_trace_level

    > 从服务器的半复制调试跟踪级别

* rpl_stop_slave_timeout

    > 设置STOP SLAVE超时之前等待的秒数
* server_uuid

    > 服务器全局唯一ID，由启动是自动生成

* show-slave-auth-info

    > 在主服务器上使用*SHOW_SLAVE HOSTS*是否可以显示用户名和密码

* simplified_binlog_gtid_recovery

    > 控制二进制日志在GTID恢复期间如何迭代

* skip-slave-start

    > 如果设置了，则从服务器不会自动启动

* slave_allow_batching

    > 为从服务器打开或关闭批量更新机制

* slave-load-tmpdir

    > slave存放临时文件的位置

* slave-skip-errors

    > 告诉从服务器线程当查询出错时继续复制

* slave-checkpoint-group｜slave_checkpoint_group

    > 在检查操作更新进程状态之前一个多线程处理的最大事务量

* slave-checkpoint-period｜slave_checkpoint_period

    > 更新进程状态及刷新中继日志到磁盘的周期

* slave_compressed_protocol

    > 使用压缩的主/从协议

* slave-max-allowed-packet|slave_max_allowed_packet

    > 从主服务器到从服务器发送的包的最大大小

* slave_net_timeout

    > 从连接中读超时时间

* slave-parallel-type

    > 告诉从服务器使用来自主服务器的数据库分区或时间缀信息处理并行事务，默认是*DATABASE*

* slave_parallel_workers

    > 并行执行事件的工作线程数，默认为0,则不启用多线程

* slave-pending-jobs-size-max

    > 从服务器工作队列持未被处理事件的最大数

* slave_pending_jobs_size_max

    > 从服务器工作队列挂起的最大作业数

* slave-rows-search-algorithms|slave_rows_search_algorithms

    > 决定用于从服务器更新的检索算法，如INDEX_SEARCH、TABLE_SCAN、HASH_SCAN；默认为TABLE_SCAN,INDEX_SCAN

* slave_transaction_retries

    > 从服务器SQL线程重试一个事务的次数

* slave_type_conversions
    
    > 控制类型转换模式在从服务器上，有ALL_LOSSY,ALL_NON_LOSSY，设置为空则不允许类型转换

* sync_binlog

    > 同步日志到磁盘

* sync_master_info

    > 同步master.info到磁盘

* sync_relay_log

    > 同步中继日志到磁盘

* sync_relay_log_info

    > 同步relay.info文件到磁盘

* binlog-checksum|binlog_checksum

    > 启用/禁用二进制日志校验

* binlog-do-db

    > 限制记录指定的数据库到二进制日志

* binlog-ignore-db

    > 不记录指定的数据库到二进制日志

* binlog_impossible_mode

    > 控制如果服务器不能写日志时该怎么处理，默认是IGNORE_ERROR，意味着服务器记录错误但继续处理更新，ABORT_SERVER则关闭服务当不能写时

* binlog-row-event-max-size

    > 二进制日志最大事件大小

* binlog_cache_size

    > 缓存中存放SQL语句的大小

* binlog_max_flush_queue_time

    > 在刷新日志之前多久读取事务

* binlog_order_commits

    > 是否以相同的顺序写入二进制日志提交

* binlog_format

    > 指定二进制日志格式

* log_bin_basename

    > 二进制日志路径及文件名

* log-bin-use-v1-row-events｜log_bin_use_v1_row_events

    > 是否使用版本1的日志行事件

* master-verify-checksum

    > 使master在遍地日志的时候进行校验

* max_binlog_size

    > 当大小超过此值，二进制日志将被自动旋转

* slave-sql-verify-checksum

    > 当从服务器读中继日志时进行检验


:corn: **复制主服务器选项和变量**

* auto_increment_increment

    > 控制连续列值之间的间隔。`auto_increment_increment`和`auto_increment_offset`使用在master到master的复制，用于控制`auto_increment`列表的操作。值在1-65535之间，默认为1。

* auto_increment_offset

    > 决定`auto_increment`列值的起始点

    ```sql
    mysql> SET @@auto_increment_offset=5;
    Query OK, 0 rows affected (0.00 sec)
    mysql> SHOW VARIABLES LIKE 'auto_inc%';
    +--------------------------+-------+
    | Variable_name | Value |
    +--------------------------+-------+
    | auto_increment_increment | 10 |
    | auto_increment_offset | 5 |
    +--------------------------+-------+
    2 rows in set (0.00 sec)
    ```

:corn: **复制从服务器选项和变量**

* log-slave-updates|--log-slave-updates

    > 一般情况下，从服务器不会记录自己的更新日志，该选项使其记录自己的更新日志到二进制日志中

* log-slow-slave-statements|--log-slow-slave-statements

    > 当慢查询日志开启时，该选项会记录从服务器上的慢查询语句

* log-warnings[=level]|--log-warnings[=level]

    > 日志错误级别，使服务器记录更多的错误信息，如果要关闭该功能则赋为0即可。

* master-info-file=file_name｜--master-info-file=file_name
    
    > 记录master信息的文件，默认叫master.info

* master-retry-count|--master-retry-count=count

    > 从服务器尝试连接主服务器的次数

* max_relay_log_size｜--max-relay-log-size=size

    > 一个中继日志文件的最大大小，超过该值则进行拆分，如果值为0则拆分由`max_binlog_size`决定

* read_only|--read_only

    > 从服务器不允许写操作除了从服务器线程及超级权限的用户

* relay-log｜--relay-log=file_name

    > 中继日志文件名，默认是host_name-relay-bin

* relay-log-index|--relay-log-index=file_name

    > 中继日志索引文件，默认host_name-relay-bin.index

* relay-log-info-file｜--relay-log-info-file=file_name

    > 记录中继日志文件信息的文件，默认relay-log.info

* relay_log_purge|--relay-log-purge={0|1}

    > 开启或关闭自动删除不再需要的中继日志

* relay-log-recovery｜--relay-log-recovery={0|1}

    > 启用在服务器启动后自动恢复中继日志。这个恢复的过程就是创建一个新的中继日志文件，初始化SQL线程在新中继日志文件中的位置和初始化I/O线程对SQL线程的位置，从主服务器读取中继日志并继续。这应该用于崩溃的服务器上确保没有被破坏的中继日志进行处理，默认为0

* relay_log_space_limit｜--relay-log-space-limit=size

    > 所有的中继日志在从服务器上占用空间大小，如果是0,则不做限制

* replicate-do-db|--replicate-do-db=db_name

    > 复制指定的数据库

* replicate-ignore-db|--replicate-ignore-db=db_name

    > 不复制指定的数据库

* replicate-do-table｜--replicate-do-table=db_name.tbl_name

    > 复制指定的表

* replicate-ignore-table｜--replicate-ignore-table=db_name.tbl_name

    > 不复制指定的表

* replicate-rewrite-db｜--replicate-rewrite-db=from_name->to_name

    > 复制更新数据库为新的名字

* replicate-wild-do-table|--replicate-wild-do-table=db_name.tbl_name

    > 复制匹配正则表达式的表

* replicate-wild-ignore-table｜ --replicate-wild-ignore-table=db_name.tbl_name

    >不复制匹配正则表达式的表

* report-host|--report-host=host_name

    > 在slave注册过程中，向master报告slave的主机名或IP

* report-password|--report-password=password

    > 向master报名账号密码

* report-port｜--report-port=slave_port_num

    > 报告连接slave的端口

* report-user｜--report-user=user_name

    > 报告slave用的用户名

* show-slave-auth-info｜--show-slave-auth-info
* slave-checkpoint-group｜--slave-checkpoint-group=#
* slave-checkpoint-period｜--slave-checkpoint-period=#
* slave-parallel-workers|--slave-parallel-workers
    > 设置并行执行复制事务的从服务器工作线程的数量

* skip-slave-start|--skip-slave-start
* slave_compressed_protocol|--slave_compressed_protocol={0|1}
* slave-load-tmpdir| --slave-load-tmpdir=file_name
* slave-max-allowed-packet|--slave-max-allowed-packet=#
* slave-net-timeout|--slave-net-timeout=seconds
* slave-parallel-type|--slave-parallel-type=type
* slave-rows-search-algorithms|slave-rows-search-algorithms=list
* slave-skip-errors|--slave-skip-errors=[err_code1,err_code2,...|all|ddl_exist_errors]
* slave-sql-verify-checksum|--slave-sql-verify-checksum={0|1}
* abort-slave-event-count|--abort-slave-event-count=#

    > 当该选项设为下数，则会如下影响复制行为：在SQL线程启动后，value日志事务被执行，之后SQL线程不会收到任何更多的事务当网络连接断开后，这时slave线程还在继续跑，但不会从中继日志读任何事务

* master-info-repository|--master-info-repository={FILE|TABLE}
* relay-log-info-repository|--relay-log-info-repository={FILE|TABLE}


:corn: **二进制日志记录选项和变量**

* binlog-row-event-max-size|--binlog-row-event-max-size=N
* log-bin|--log-bin[=base_name]

    > 启用二进制日志记录，服务器记录所有的修改数据的语句到二进制日志中，用来备份和复制。base_name指定日志文件名字

* log-bin-index｜--log-bin-index[=file_name]

    > 日志索引文件名

* log-bin-use-v1-row-events｜--log-bin-use-v1-row-events[={0|1}]
* log-short-format｜--log-short-format
    
    > 记录少量的信息到二进制日志和慢查询日志中

* binlog-do-db｜--binlog-do-db=db_name
* binlog-ignore-db｜--binlog-ignore-db=db_name
* binlog-checksum｜--binlog-checksum={NONE|CRC32}
* master-verify-checksum｜--master-verify-checksum={0|1}
* max-binlog-dump-events｜--max-binlog-dump-events=N

    > 此选项被mysql测试套件用于内部复制测试和调试

* binlog-rows-query-log-events｜--binlog-rows-query-log-events

    > 该选项打开binlog_rows_query_log_events

* binlog_cache_size|--binlog_cache_size=#
* binlog_checksum
* binlog-format|--binlog-format=format

    > 该选项设置二进制日志格式，可以是基于语句的、行的或混合

* binlog_impossible_mode|--binlog_impossible_mode[=value]
* binlog_order_commits
* log_bin

    > 是否开启二进制日志

* log_bin_basename
* log_bin_index

    > 二进制日志索引文件

* log_bin_use_v1_row_events|--log-bin-use-v1-row-events[={0|1}]
* log_slave_updates|--log-slave-updates
* master_verify_checksum

    > 开启主服务器检查校验当读取二进制日志时

* max_binlog_cache_size｜--max_binlog_cache_size=#

    > 仅设置事务缓存大小

* max_binlog_size
* sync_binlog

 
## 复制的实现


## 复制的解决方案

## 常用命令

* `stop slave`    停止slave
* `start slave`    启动slave
* `reset slave`    重制slave

## 复制同步有以下三个线程实现：

1. 在master上的二进制日志转储线程(binlog dump thread)
1. 在slave上的I/O线程(slave i/o thread)
1. 在slave上的sql线程(slave SQL thread) 

## 主服务器配置样例

```sql
# binary logging is required for replication
log-bin=mysql-bin
# required unique id between 1 and 2^32 - 1
# defaults to 1 if master-host is not set
# but will not function as a master if omitted
server-id   = 1
#忽略复制的数据库
binlog-ignore-db=mysql,information_schema,performance_schema
#binlog-row-event-max-size = 8192
#log-bin-index = dbl-wise-rdtest10-xxx-com-log-bin.index
#只复制的数据库
binlog-do-db = test
# binary logging format - mixed recommended
binlog_format=mixed
```

## 从服务器配置样例

```sql
[mysqld]
server-id=10

log-bin=mysql-bin
#设置基于行复制的最大数
#binlog-row-event-max-size = 8192
#log-bin-index = fedev-xxx-com-log-bin.index

# required unique id between 1 and 2^32 - 1
# defaults to 1 if master-host is not set
# but will not function as a master if omitted
server-id   = 10
log-slave-updates
#master-info-file = master.info
#master-retry-count = 86400
#max_relay_log_size = 0
read_only
relay-log = fedev-xxx-com-relay-bin
#relay-log-index = fedev-xxx-com-relay-bin.index
#relay-log-info-file = relay-log.info
relay_log_purge = 1
relay-log-recovery = 1
#slave上relay-log的最大上限，0表示没有限制
#relay_log_space_limit = 0
#忽略不复制哪些数据库
replicate-ignore-db = mysql,information_schema,performance_schema
#只复制哪些数据库
replicate-do-db = test
#向master汇报显示的register的slave的host
report-host = fedev.xxx.com
#向master汇报显示的register的slave连接密码
report-password = 123456
#汇报显示username
report-user = root
#显示slave的权限信息
show-slave-auth-info
#服务启动时不启动slave,即手动启动slave,start slave;
skip-slave-start
#只复制哪些表
#replicate-do-table = user
#忽略不复制哪些表
#replicate-ignore-table = othertable
#重新命名
#replicate-rewrite-db = oldname->newname
#复制正则匹配的数据库及表
#replicate-wild-do-table = test%.user%
#忽略不复制正则匹配的数据库及表
#replicate-wild-ignore-table = test%.user%
#创建临时文件的目录
#slave-load-tmpdir = /tmp
#该选项设置的最大数据包大小为slave和master
#slave-max-allowed-packet =  1073741824
#数据master传到slave的连接超时
#slave-net-timeout = 3600
#slave批处理扫描方式
#slave-rows-search-algorithms = TABLE_SCAN,INDEX_SCAN
#跳过slave错误提示
#slave-skip-errors = all,1062,1053
#开启slave批处理
#slave_allow_batching
```
