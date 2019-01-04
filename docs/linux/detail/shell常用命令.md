* basename

    ```bash
    basename path
    ```

    > 提取路径中的文件名

* compress|uncompress

    ```bash
    compress [-fv] [-b bits] [file ...]
    compress -c [-b bits] [file]
    uncompress [-fv] [file ...]
    uncompress -c [file]
    ```

    > 压缩文件，压缩后的文件名具有.z后缀，也可以解压文件

* diff

    ```bash
    diff [OPTION]... FILES
    ```

    * -c 按照标准格式输出
    * -i 忽略大小写

    > 显示出两个文件中不一致的行

* dirname

    > 显示目录的路径部分，与basename相反

* du

    `du options dir`

    * -a    显示每个文件的大小，不仅是整个目录所占用的空间
    * -s    只显示总计

    `du -sh *` 查看当前目录以下搜索文件和子目录大小

* df

    `df -h`

    > 查看磁盘各分区大小、已用空间等信息

* file

    `file filename`

    > 用来确定文件的类型

* fuser

    `fuser options file`

    * -k 杀死所有访问该文件或文件系统的进程
    * -u 显示访问该文件或文件系统的所有进程

    > 显示访问某个文件或文件系统的所有的进程。在有些系统上-u和-m选项可以互换

* logname

    > 该命令可以显示当前所有使用的登录用户

* nl

    `nl options file`

    * -l    行号每次增加n；缺省为1
    * -p    在新的一页不重新计数
    
    > nl命令可以用于在文件中列行号，在打印源代码或列日志文件时很有用

* sleep

    > 该命令使用系统等待相应的秒数，如`sleep 10`

* strings

    > 该命令可以看二进制文件中所包含的文本

* tty

    > 可以使用tty来报告所连接的设备或终端,可以使用tty -s命令来确定脚本的标准输入，返回码为0-终端，1-非终端

    ```bash
    $tty
    /dev/ttys001
    ```  

* uname

    > 显示当前操作系统名及其他相关信息

    * -a    显示所有信息
    * -s    显示系统名
    * -v    只显示操作系统版本或其发布日期

    ```bash
    $uname -a
    Darwin baidumacpro.local 15.3.0 Darwin Kernel Version 15.3.0: Thu Dec 10 18:40:58 PST 2015; root:xnu-3248.30.4~1/RELEASE_X86_64 x86_64
    $uname -v
    Darwin Kernel Version 15.3.0: Thu Dec 10 18:40:58 PST 2015; root:xnu-3248.30.4~1/RELEASE_X86_64
    ```

* wait

    `wait process id`

    > 该命令可以用来等待进程号为processid的进程或所有的后台进程结束后，再执行当前脚本


* wc

    `wc options files`

    * -c    显示字符数
    * -l    显示行数
    * -w    显示单词数

    > 统计文件中的字符数、单词数和行数

* whereis

    `whereis comman`

    > 给出系统命令的二进制文件及其在线手册的路径

    ```bash
    $whereis fuser
    /usr/bin/fuser
    ```

* who

    `who options`

    > 显示当前有哪些用户登录到系统上。whoami显示执行该命令的用户名。

    * -a    显示所有的结果
    * -r    显示当前的运行级别
    * -s    列出用户名及时间域

* alias

    ```bash
    usage: alias [-p] [name[=value] ... ]
    ```

    > 设置指令别名，如`alias ll='ls -l'`，若不加任何参数，则可以列出所有的指令别名设置

