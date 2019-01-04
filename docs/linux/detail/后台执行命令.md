## crontab命令

crontab 是用来让使用者在固定时间或固定间隔执行程序之用，换句话说，也就是类似使用者的时程表。

```bash
crontab [-u user] file
crontab [-u user] { -l | -r | -e }
```

其中：

* -u 用户名,如果使用自己的名字登录，就不用使用-u选项，因为该命令能够知道当前的用户
* -e 编辑crontab文件
* -l 列出crontab文件中的内容
* -r 删除crontab文件

### crontab格式

`分 时 日 月 星期 命令`

* 第1列 分钟1~59
* 第2列 小时1~23（0表示子夜）
* 第3列 日1~31
* 第4列 月1~12
* 第5列 星期0~6（0表示星期天）
* 第6列 要运行的命令

这些域中可以用以下符号：

* - 表示一个时间范围，如星期一到星期五，`1-5`
* , 表示列举，如星期一和星期四，`1,4`
* * 表示连续的时间段，如果你对某个表示时间的域没有特别限定，也应该在该域填入*

### 创建crontab

```bash
vim democron
#输入 0,15,30,45 18-06 * * * /bin/echo `date` > /dev/console ,并保存
crontab democron

#也可以直接执行下面命令创建输入
crontab -e
```

### 编辑修改crontab

```bash
crontab -e
```

### 列出crontab文件内容

```bash
crontab -l
```

### 删除crontab文件

```bash
crontab -r
```

## at命令

at命令允许用户向cron守护进程提交作业，使其在稍后的时间运行。这里稍后的时间可能是10分钟，也可能是几天之后，如果需要在更长的时间后运行，则最好使用crontab。该作业的所有输出都将以电子邮件的形式发送给用户。

语法：

```bash
at [-f script] [-m -l -r] [time]
```

其中：

* -f script 是所要提交的脚本或命令
* -l 列出当前所有等待运行的作业，atq命令具有相同的作用
* -r 清除作业，需要提供作业标识ID，有些要使用atrm命令清除
* -m 作业完成后指定给用户发邮件
* time at命令的时间格式，可以是：

    * 标准的小时和分钟格式，比如：10：15；
    * ~AM/~PM指示符，比如：10：15~PM;
    * 特定可命名的时间，比如：now、noon、midnight、teatime
    * 标准日期格式，比如MMDDYY、MM/DD/YY,DD.MM.YY;
    * 文本日期，比如：jul 4, Dec 25，加不加年份均可
    * 增量时间：
        * 当前时间+25min
        * 明天10：15~PM;
        * 10:15+7天

### 使用at提交命令或脚本

一般需要提交多行命令时，使用at命令提示符方式，而在提交shell脚本时，使用命令行方式，如果是命令提示符方式，则在最后按CTRL-D退出。

```bash
$ at 15:50
find / -name 'passwd' -print
job 2 at Tue Mar  1 15:50:00 2016
```

或是

```bash
echo find /etc -name 'passwd' -print | at now +1 minute
```

### 列出所有提交的作业

```bash
at -l
```

### 清除指定作业

```bash
at -r 1
#或是
atrm 1
```

## &命令

当需要终端来做别的交互事情时，可以通过以下形式把命令放在后台运行：

```bash
命令 &
```

作业会将结果一样输出到屏幕，如果输出量太的话，可以通过下面形式进行输出重定向：

```bash
command > out.file 2>&1 &
```

这样会把所有的标准输出和错误输出都重定向到一个叫做out.file的文件中

## nohup命令

如果正在运行一个进程，而在退出账户时该进程还不会结束，那么可以用nohup命令。该命令可以在你退出账户之后续续运行相应的进程。

```bash
nohup command &
```

如果使用nohup命令提交作业，那么在缺省情况下该作业的所有输出都被重定向到一个名为nohup.out的文件中，除非另外指定了输出文件：

```bash
nohup command > myout.file 2>&1
```

如果一次需要提交多个命令，则最好把他们写到一个shell脚本中，并用nohup命令来执行它，如下所示：

```bash
echo cat /home/ip.txt | sort > test
nohup ./test > out.file 2>&1 &
```


