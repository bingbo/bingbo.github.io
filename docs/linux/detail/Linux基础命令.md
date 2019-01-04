## Shell常用命令

#### cd命令：

切换当前目录命令，参数可以是绝对路径或相对路径。如：

```bash
cd /root/Docements # 切换到目录/root/Docements  
cd ./path          # 切换到当前目录下的path目录中，“.”表示当前目录    
cd ../path         # 切换到上层目录中的path目录中，“..”表示上一层目录  
```

#### ls命令：

查看文件与目录的列表命令，常用的参数如下：

```bash
-l ：列出长数据串，包含文件的属性与权限数据等  
-a ：列出全部的文件，连同隐藏文件（开头为.的文件）一起列出来（常用）  
-d ：仅列出目录本身，而不是列出目录的文件数据  
-h ：将文件容量以较易读的方式（GB，kB等）列出来  
-R ：连同子目录的内容一起列出（递归列出），等于该目录下的所有文件都会显示出来  
```

注：这些参数也可以组合使用，例如：

```bash
ls -l #以长数据串的形式列出当前目录下的数据文件和目录  
ls -lR #以长数据串的形式列出当前目录下的所有文件 
```

#### grep命令：

用于分析文本文件一行的信息，该命令通常与管道命令一起使用。通常用于对一些文本行进行筛选等，如：

```bash
grep [-acinv] [--color=auto] '查找字符串' filename  
```

常用参数如下：

```bash
-a ：将binary文件以text文件的方式查找数据  
-c ：计算找到‘查找字符串’的次数  
-i ：忽略大小写的区别，即把大小写视为相同  
-v ：反向选择，即显示出没有‘查找字符串’内容的那一行  
# 例如：
# 取出文件/etc/man.config中包含MANPATH的行，并把找到的关键字加上颜色  
grep --color=auto 'MANPATH' /etc/man.config  
# 把ls -l的输出中包含字母file（不区分大小写）的内容输出  
ls -l | grep -i file  
```

#### find命令：

查找文件的命令，基本语法如下：

```bash
find [PATH] [option] [action]
```

与时间相关的参数：

```bash
-mtime n ：n为数字，意思为在n天之前的“一天内”被更改过的文件；  
-mtime +n ：列出在n天之前（不含n天本身）被更改过的文件名；  
-mtime -n ：列出在n天之内（含n天本身）被更改过的文件名；  
-newer file ：列出比file还要新的文件名  
# 例如：  
find /root -mtime 0 # 在当前目录下查找今天之内有改动的文件  
```

与用户或用户组名相关的参数：

```bash
-user name ：列出文件所有者为name的文件  
-group name ：列出文件所属用户组为name的文件  
-uid n ：列出文件所有者为用户ID为n的文件  
-gid n ：列出文件所属用户组为用户组ID为n的文件  
# 例如：  
find /home/ljianhui -user ljianhui # 在目录/home/ljianhui中找出所有者为ljianhui的文件
```

与文件权限及名称相关的参数：

```bash
-name filename ：找出文件名为filename的文件  
-size [+-]SIZE ：找出比SIZE还要大（+）或小（-）的文件  
-tpye TYPE ：查找文件的类型为TYPE的文件，TYPE的值主要有：一般文件（f)、设备文件（b、c）、  
             目录（d）、连接文件（l）、socket（s）、FIFO管道文件（p）；  
-perm mode ：查找文件权限刚好等于mode的文件，mode用数字表示，如0755；  
-perm -mode ：查找文件权限必须要全部包括mode权限的文件，mode用数字表示  
-perm +mode ：查找文件权限包含任一mode的权限的文件，mode用数字表示  
# 例如：  
find / -name passwd # 查找文件名为passwd的文件  
find . -perm 0755 # 查找当前目录中文件权限的0755的文件  
find . -size +12k # 查找当前目录中大于12KB的文件，注意c表示byte  
```

#### cp命令：

用于复制文件或目录，常用参数如下：

```bash
-a ：将文件的特性一起复制  
-p ：连同文件的属性一起复制，而非使用默认方式，与-a相似，常用于备份  
-i ：若目标文件已经存在时，在覆盖时会先询问操作的进行  
-r ：递归持续复制，用于目录的复制行为  
-u ：目标文件与源文件有差异时才会复制  
# 例如 ：

cp -a file1 file2 #连同文件的所有特性把文件file1复制成文件file2  
cp file1 file2 file3 dir #把文件file1、file2、file3复制到目录dir中  
```

#### mv命令：

用于移动文件、目录或更名，常用参数如下：

```bash
-f ：force强制的意思，如果目标文件已经存在，不会询问而直接覆盖  
-i ：若目标文件已经存在，就会询问是否覆盖  
-u ：若目标文件已经存在，且比目标文件新，才会更新  
# 注：该命令可以把一个文件或多个文件一次移动一个文件夹中，但是最后一个目标文件一定要是“目录”。
# 例如：

mv file1 file2 file3 dir # 把文件file1、file2、file3移动到目录dir中  
mv file1 file2 # 把文件file1重命名为file2  
```

#### rm命令：

用于删除文件或目录，常用参数如下：

```bash
-f ：就是force的意思，忽略不存在的文件，不会出现警告消息  
-i ：互动模式，在删除前会询问用户是否操作  
-r ：递归删除，最常用于目录删除，它是一个非常危险的参数  
# 例如：

rm -i file # 删除文件file，在删除之前会询问是否进行该操作  
rm -fr dir # 强制删除目录dir中的所有文件
```

#### ps命令：

用于输出进程运行状况，常用参数如下：

```bash
-A ：所有的进程均显示出来  
-a ：不与terminal有关的所有进程  
-u ：有效用户的相关进程  
-x ：一般与a参数一起使用，可列出较完整的信息  
-l ：较长，较详细地将PID的信息列出  
```

常用命令参数，如下：

```bash
ps aux # 查看系统所有的进程数据  
ps ax # 查看不与terminal有关的所有进程  
ps -lA # 查看系统所有的进程数据  
ps axjf # 查看连同一部分进程树状态  
```

#### kill命令：

用于向某个工作（%jobnumber）或者是某个PID（数字）传送一个信号，通常与ps和jobs命令一起使用，基本语法如下：

```bash
kill -signal PID  
```

signal的常用参数如下： 

注：最前面的数字为信号的代号，使用时可以用代号代替相应的信号。

```bash
1 ：SIGHUP，启动被终止的进程  
2 ：SIGINT，相当于输入ctrl+c，中断一个程序的进行  
9 ：SIGKILL，强制中断一个进程的进行  
15：SIGTERM，以正常的结束进程方式来终止进程  
17：SIGSTOP，相当于输入ctrl+z，暂停一个进程的进行  
# 例如：

# 以正常的结束进程方式来终于第一个后台工作，可用jobs命令查看后台中的第一个工作进程  
kill -SIGTERM %1   
# 重新改动进程ID为PID的进程，PID可用ps命令通过管道命令加上grep命令进行筛选获得  
kill -SIGHUP PID  
```

#### killall命令：

用于向一个命令启动的进程发送一个信号，语法如下：

```bash
killall [-iIe] [command name] 
```

参数如下：

```bash
-i ：交互式的意思，若需要删除时，会询问用户  
-e ：表示后面接的command name要一致，但command name不能超过15个字符  
-I ：命令名称忽略大小写  
# 例如：  
killall -SIGHUP syslogd # 重新启动syslogd  
```

#### file命令：

用于输出文件的基本数据。在Linux下文件的无后缀，可以通过此命令获取文件类型等信息。语法如下：

```bash
file filename  
# 例如：  
file ./test  
```

#### tar命令：

用于对文件进行打包，默认情况并不会压缩。需要指定相应的参数，才会调用相应的压缩程序（如gzip和bzip等）进行压缩和解压。

参数如下：

```bash
-c ：新建打包文件  
-t ：查看打包文件的内容含有哪些文件名  
-x ：解打包或解压缩的功能，可以搭配-C（大写）指定解压的目录，注意-c,-t,-x不能同时出现在同一条命令中  
-j ：通过bzip2的支持进行压缩/解压缩  
-z ：通过gzip的支持进行压缩/解压缩  
-v ：在压缩/解压缩过程中，将正在处理的文件名显示出来  
-f filename ：filename为要处理的文件  
-C dir ：指定压缩/解压缩的目录dir  
```

常用命令如下：

```bash
压缩：tar -jcv -f filename.tar.bz2 要被处理的文件或目录名称  
查询：tar -jtv -f filename.tar.bz2  
解压：tar -jxv -f filename.tar.bz2 -C 欲解压缩的目录  
# 注：文件名并不定要以后缀tar.bz2结尾，这里主要是为了说明使用的压缩程序为bzip2
```

#### cat命令：

用于查看文本文件的内容。通常可用管道与more和less一起使用，可以分页查看数据。例如：

```bash
cat text | less # 查看text文件中的内容  
# 注：这条命令也可以使用less text来代替  
```

#### chgrp命令：

用于改变文件所属用户组，基本用法如下：

```bash
chgrp [-R] dirname/filename  
-R ：进行递归的持续对所有文件和子目录更改  
# 例如：  
chgrp users -R ./dir # 递归地把dir目录下中的所有文件和子目录下所有文件的用户组修改为users  
```

#### chown命令：

用于改变文件的所有者，与chgrp命令的使用方法相同，只是修改的文件属性不同，不再详述。

#### chmod命令：

用于改变文件的权限，一般的用法如下：

```bash
chmod [-R] xyz 文件或目录  
```

-R：进行递归的持续更改，即连同子目录下的所有文件都会更改  
同时，chmod还可以使用u（user）、g（group）、o（other）、a（all）和+（加入）、-（删除）、=（设置）跟rwx搭配来对文件的权限进行更改。

```bash
# 例如：  
chmod 0755 file # 把file的文件权限改变为-rxwr-xr-x  
chmod g+w file # 向file的文件权限中加入用户组可写权限  
```

#### vim命令：

要用于文本编辑，它接一个或多个文件名作为参数，如果文件存在就打开，如果文件不存在就以该文件名创建一个文件。vim是一个非常好用的文本编辑器，它里面有很多非常好用的命令。

#### time命令：

用于计算一个命令（程序）的执行时间。使用简单，在命令的前面加一个time即可，例如：

```bash
time ./process  
time ps aux  
```

在程序或命令运行结束后，在最后输出了三个时间，它们分别是：

```bash
user：用户CPU时间，命令执行完成花费的用户CPU时间，即命令在用户态中执行时间总和；
system：系统CPU时间，命令执行完成花费的系统CPU时间，即命令在核心态中执行时间总和；
real：实际时间，从command命令行开始执行到运行终止的消逝时间；
```

注：用户CPU时间和系统CPU时间之和为CPU时间，即命令占用CPU执行的时间总和。实际时间要大于CPU时间，因为Linux是多任务操作系统，往往在执行一条命令时，系统还要处理其它任务。另一个需要注意的问题是即使每次执行相同命令，但所花费的时间也是不一样，其花费时间是与系统运行相关的


## awk命令

AWK是一种处理文本文件的语言，是一个强大的文本分析工具。
之所以叫AWK是因为其取了三位创始人 Alfred Aho，Peter Weinberger，和 Brian Kernighan 的Family Name 的首字符。

语法：

```bash
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```

选项参数说明：

```bash
-F fs or --field-separator fs
指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:。
-v var=value or --asign var=value
赋值一个用户定义变量。
-f scripfile or --file scriptfile
从脚本文件中读取awk命令。
-mf nnn and -mr nnn
对nnn值设置内在限制，-mf选项限制分配给nnn的最大块数目；-mr选项限制记录的最大数目。这两个功能是Bell实验室版awk的扩展功能，在标准awk中不适用。
-W compact or --compat, -W traditional or --traditional
在兼容模式下运行awk。所以gawk的行为和标准的awk完全一样，所有的awk扩展都被忽略。
-W copyleft or --copyleft, -W copyright or --copyright
打印简短的版权信息。
-W help or --help, -W usage or --usage
打印全部awk选项和每个选项的简短说明。
-W lint or --lint
打印不能向传统unix平台移植的结构的警告。
-W lint-old or --lint-old
打印关于不能向传统unix平台移植的结构的警告。
-W posix
打开兼容模式。但有以下限制，不识别：/x、函数关键字、func、换码序列以及当fs是一个空格时，将新行作为一个域分隔符；操作符<strong>和</strong>=不能代替^和^=；fflush无效。
-W re-interval or --re-inerval
允许间隔正则表达式的使用，参考(grep中的Posix字符类)，如括号表达式[[:alpha:]]。
-W source program-text or --source program-text
使用program-text作为源代码，可与-f命令混用。
-W version or --version
打印bug报告信息的版本。
```

基本用法：

log.txt文本内容如下：

```bash
2 this is a test
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```

格式化输出：

```bash
awk '{[pattern] action}' {filenames}   # 行匹配语句 awk '' 只能用单引号
```

实例：

```bash
# 每行按空格或TAB分割，输出文本中的1、4项
 $ awk '{print $1,$4}' log.txt
 ---------------------------------------------
 2 a
 3 like
 This's
 10 orange,apple,mongo
 # 格式化输出
 $ awk '{printf "%-8s %-10s\n",$1,$4}' log.txt
 ---------------------------------------------
 2        a
 3        like
 This's
 10       orange,apple,mongo
 ```

指定分隔符：

```bash
awk -F  #-F相当于内置变量FS, 指定分割字符
```

实例：

```bash
# 使用","分割
 $  awk -F, '{print $1,$2}'   log.txt
 ---------------------------------------------
 2 this is a test
 3 Are you like awk
 This's a test
 10 There are orange apple
 # 或者使用内建变量
 $ awk 'BEGIN{FS=","} {print $1,$2}'     log.txt
 ---------------------------------------------
 2 this is a test
 3 Are you like awk
 This's a test
 10 There are orange apple
 # 使用多个分隔符.先使用空格分割，然后对分割结果再使用","分割
 $ awk -F '[ ,]'  '{print $1,$2,$5}'   log.txt
 ---------------------------------------------
 2 this test
 3 Are awk
 This's a
 10 There apple
 ```

设置变量：

```bash
awk -v  # 设置变量
```

实例：

```bash
 $ awk -va=1 '{print $1,$1+a}' log.txt
 ---------------------------------------------
 2 3
 3 4
 This's 1
 10 11
 $ awk -va=1 -vb=s '{print $1,$1+a,$1b}' log.txt
 ---------------------------------------------
 2 3 2s
 3 4 3s
 This's 1 This'ss
 10 11 10s
 ```

执行AWK脚本：

```bash
awk -f {awk脚本} {文件名}
```

实例：

```bash
 $ awk -f cal.awk log.txt
 ```

## sed命令

sed 是一种在线编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（Pattern Space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等。

#### sed使用参数：

```bash
[root@www ~]# sed [-nefr] [动作]
```

#### 选项与参数：

```bash
-n ：使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到终端上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。
-e ：直接在命令列模式上进行 sed 的动作编辑；
-f ：直接将 sed 的动作写在一个文件内， -f filename 则可以运行 filename 内的 sed 动作；
-r ：sed 的动作支持的是延伸型正规表示法的语法。(默认是基础正规表示法语法)
-i ：直接修改读取的文件内容，而不是输出到终端。
动作说明： [n1[,n2]]function
n1, n2 ：不见得会存在，一般代表『选择进行动作的行数』，举例来说，如果我的动作是需要在 10 到 20 行之间进行的，则『 10,20[动作行为] 』
function：
a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
p ：列印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！
```

#### 以行为单位的新增/删除：

将 /etc/passwd 的内容列出并且列印行号，同时将第2~5行删除。

```bash
[root@www ~]# nl /etc/passwd | sed '2,5d'
1 root:x:0:0:root:/root:/bin/bash
6 sync:x:5:0:sync:/sbin:/bin/sync
7 shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
```

sed 的参数为'2,5d'，那个d就是删除，所以显示的数据就没有2~5行。另外，注意原本应该是要下达sed -e 才对，没有 -e 也行。同时也要注意的是，sed后面接的动作，请务必以''两个单引号括住。

#### 删除第二行

```bash
nl /etc/passwd | sed '2d' 
```

#### 删除第3到最后一行

```bash
nl /etc/passwd | sed '3,$d' 
```

#### 在第二行后加上『drink tea?』字样

```bash
[root@www ~]# nl /etc/passwd | sed '2a drink tea'
1 root:x:0:0:root:/root:/bin/bash
2 bin:x:1:1:bin:/bin:/sbin/nologin
drink tea
3 daemon:x:2:2:daemon:/sbin:/sbin/nologin
.....(后面省略).....
```

#### 在第二前加上『drink tea?』字样 

```bash 
nl /etc/passwd | sed '2“i” drink tea'
```

#### 如果是要增加两行以上，在第二行后面加入两行字，例如『Drink tea or .....』与『drink beer?』  

```bash
[root@www ~]# nl /etc/passwd | sed '2a Drink tea or ......\

drink beer ?'
1 root:x:0:0:root:/root:/bin/bash
2 bin:x:1:1:bin:/bin:/sbin/nologin
Drink tea or ......
drink beer ?
3 daemon:x:2:2:daemon:/sbin:/sbin/nologin
```

每一行之间都必须要以反斜杠『\』来进行新行的添加。所以上面的例子中，在第一行的最后面就有\存在。  

#### 以行为单位的替换与显示：

将第2~5行的内容替换为『No 2-5 number』呢？

```bash
[root@www ~]# nl /etc/passwd | sed '2,5c No 2-5 number'
1 root:x:0:0:root:/root:/bin/bash
No 2-5 number
6 sync:x:5:0:sync:/sbin:/bin/sync
```

透过这个方法我们就能够将数据整行替换了。

#### 仅列出/etc/passwd文件内的5~7行

```bash
[root@www ~]# nl /etc/passwd | sed -n '5,7p'
5 lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
6 sync:x:5:0:sync:/sbin:/bin/sync
7 shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
```

可以通过这个sed的打印功能，就能够将某一个文件内的某些行号选择出来显示。

#### 数据的搜寻并显示：

搜索 /etc/passwd有root关键字的行

```bash
nl /etc/passwd | sed '/root/p'
1  root:x:0:0:root:/root:/bin/bash
1  root:x:0:0:root:/root:/bin/bash
2  daemon:x:1:1:daemon:/usr/sbin:/bin/sh
3  bin:x:2:2:bin:/bin:/bin/sh
4  sys:x:3:3:sys:/dev:/bin/sh
5  sync:x:4:65534:sync:/bin:/bin/sync
```

如果root找到，除了输出所有行，还会输出匹配行。

#### 使用-n的时候将只打印包含模板的行。

```bash
nl /etc/passwd | sed -n '/root/p'
1  root:x:0:0:root:/root:/bin/bash
```

#### 数据的搜寻并删除：

删除/etc/passwd所有包含root的行，其他行输出

```bash
nl /etc/passwd | sed  '/root/d'
2  daemon:x:1:1:daemon:/usr/sbin:/bin/sh
3  bin:x:2:2:bin:/bin:/bin/sh
```

第一行的匹配root已经删除了

#### 数据的搜寻并执行命令：

找到匹配模式eastern的行后，搜索/etc/passwd，找到root对应的行，执行后面花括号中的一组命令，每个命令之间用分号分隔，这里把bash替换为blueshell，再输出这行：

```bash
 nl /etc/passwd | sed -n '/root/{s/bash/blueshell/;p}'
 1  root:x:0:0:root:/root:/bin/blueshell
 ```

#### 如果只替换/etc/passwd的第一个bash关键字为blueshell，就退出：

```bash
nl /etc/passwd | sed -n '/bash/{s/bash/blueshell/;p;q}'    
1  root:x:0:0:root:/root:/bin/blueshell
```

最后的q是退出。

#### 数据的搜寻并替换：

除了整行的处理模式之外， sed还可以用行为单位进行部分数据的搜寻并替换。基本上sed的搜寻与替换的与vi类似：

```bash
sed 's/要被取代的字串/新的字串/g'
```

#### 先观察原始信息，利用/sbin/ifconfig查询IP

```bash
[root@www ~]# /sbin/ifconfig eth0
eth0 Link encap:Ethernet HWaddr 00:90:CC:A6:34:84
inet addr:192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
inet6 addr: fe80::290:ccff:fea6:3484/64 Scope:Link
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
```

本机的IP是192.168.1.100。

#### 将IP前面的部分删除

```bash
[root@www ~]# /sbin/ifconfig eth0 | grep 'inet addr' | sed 's/^.*addr://g'
192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
接下来则是删除后续的部分，即：192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
```

#### 将IP后面的部分删除

```bash
[root@www ~]# /sbin/ifconfig eth0 | grep 'inet addr' | sed 's/^.*addr://g' | sed 's/Bcast.*$//g'
192.168.1.100
```

#### 多点编辑：

一条sed命令，删除/etc/passwd第三行到末尾的数据，并把bash替换为blueshell

```bash
nl /etc/passwd | sed -e '3,$d' -e 's/bash/blueshell/'
1  root:x:0:0:root:/root:/bin/blueshell
2  daemon:x:1:1:daemon:/usr/sbin:/bin/sh
```

-e表示多点编辑，第一个编辑命令删除/etc/passwd第三行到末尾的数据。第二条命令搜索bash替换为blueshell。

#### 直接修改文件内容(危险动作)：

sed可以直接修改文件的内容，不必使用管道命令或数据流重导向。但是这个动作会直接修改到原始的文件，所以请你千万不要随便拿系统配置来测试！我们还是使用下载的regular_express.txt文件来测试。

#### 利用sed将regular_express.txt内每一行结尾若为“.”则换成“!”

```bash
[root@www ~]# sed -i 's/\.$/\!/g' regular_express.txt
```

#### 利用sed在regular_express.txt最后一行加入『# This is a test』

```bash
[root@www ~]# sed -i '$a # This is a test' regular_express.txt
```

由於$代表的是最后一行，而a的动作是新增，因此该文件最后新增『# This is a test』！

sed 的『 -i 』参数可以直接修改文件内容，这功能非常有帮助！举例来说，如果你有一个100万行的文件，你要在第100行加某些文字，此时使用vim可能会疯掉，因为文件太大。利用sed直接修改/替换的功能，可以很轻松实现