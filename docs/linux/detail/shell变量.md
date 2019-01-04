### 设置变量

```bash
var = val
```

> 如果取值包含有空格，则必须用双引号括起来

### 显示变量

```bash
echo ${name}
#或
echo "hello, $name"
```

### 清除变量

```bash
name = 'bill'
unset name
```

### 显示所有本地定义的shell变量

```bash
$set
Apple_PubSub_Socket_Render=/private/tmp/com.apple.launchd.cgO3yzgLRL/Render
BASH=/bin/bash
BASH_ARGC=()
BASH_ARGV=()
```

### 结合变量值

```bash
echo ${var1}${var2}
```

### 变量是否已设置

```bash
${var:-value}

$name=bill
$echo "hello,${name:-bing}"
hello,bill
$echo "hello,${name1:-bing}"
hello,bing

##上面没有实际赋值，如果没有则实际赋值，如下
${var:=value}

##如果没有赋值，返回系统错误信息，如下
${var:?}

##或给出相应的提示
${var:?'sorry cannot read the var'}

##如果未设置返回空串，如下
${var:+val}
```

> 如果设置了变量var则使用其值，如果未设置则使用新值value。

### 只读变量

```bash
name = bill
readonly name

##查看只读变量
$readonly
declare -ar BASH_VERSINFO='([0]="3" [1]="2" [2]="57" [3]="1" [4]="release" [5]="x86_64-apple-darwin15")'
declare -ir EUID="502"
declare -ir PPID="5071"
declare -r SHELLOPTS="braceexpand:emacs:hashall:histexpand:history:interactive-comments:monitor"
declare -ir UID="502"
```

> 只读变量只需要在前面用readonly标识，如果有人包括本人在内要改变其值，则返回错误

## 环境变量

环境变量用于所有用户进程（经常称为子进程）。登录进程称为父进程。shell中执行的用户进程均称为子进程。

环境变量可在命令行设置，但用户注销时值将丢失，因此最好在.bash_profile或.profile中设置，其中的值在每次登录时会初始化。一般所有的环境变量都为大写。且用于用户进程前，必须用export命令导出。

### 设置环境变量

```bash
VAR = vale
export VAR
```

### 显示环境变量

```bash
echo $VAR

#查看所有环境变量
evn
TERM_PROGRAM=iTerm.app
SHELL=/bin/bash
TERM=linux
CLICOLOR=1
```

### 清除环境变量

```bash
unset VAR
```

### 内嵌shell变量

1. CDPATH

    > 改变目录路径变量，如果设置了CDPATH，cd一个目录时，首先查找CDPATH，如果CDPATH指明了此目录，则此目录成为当前工作目录。

    ```bash
    CDPATH=:/home/work:/user/work
    export CDPATH
    ```

2. EXINIT

    > 该变量保存使用vi编辑器时的初始化选项。如调用vi时显示行号且在第10个空格加入tab键：

    ```bash
    EXINIT='set nu tab 10'
    export
    ```

3. HOME

    > 用户的主目录

4. IFS

    > shell指定的缺省域分隔符，要设置其返回初始设置：

    ```bash
    IFS=<space><tab>
    export IFS
    ```

5. LOGNAME

    > 此变量保存登录名，应该为缺省设置，如果没设置，则可以如下完成：

    ```bash
    LOGNAME=`whoami`
    export
    ```

6. MAIL

    > 该变量保存邮箱路径名，缺省为/var/spoolmail/<login name>·

7. MAILPATH

    > 如果 有多个邮箱要用天MAILPATH，此变量设置将覆盖MAIL设置

8. PATH

    > PATH环境变量保存命令查找的目录顺序

9. PS1

    > 基本提示符包含shell提示符，缺省对超级用户为#，其他为$

    ```bash
    echo $PS1
    $
    PS1 = '->'
    export PS1
    ```

10. PS2

    > PS2为附属提示符，缺省为符号>.PS2用于执行多行命令或超过一行的一个命令

11. SHELL

    > SHELL变量保存缺省shell

    ```bash
    $echo $SHELL
    /bin/bash
    ```

12. TERMINFO

    > 终端初始化变量保存终端配置文件的位置。通常在/user/lib/terminfo或/usr/share/terminfo

13. TERM

    > 保存终端类型，设置TERM使应用获知终端对屏幕和键盘开头一人控制序列类型

    ```bash
    $echo $TERM
    linux
    ```

14. TZ

    > 时区变量保存时区值，只有系统管理员才可以更改此设置

15. EDITOR

    > 设置编辑器器，最常用

    ```bash
    EDITOR=vi
    export EDDITOR
    ```

16. PWD

    > 当前目录

17. PAGER

    > 保存屏幕翻页命令，如pg、more

18. MANPATH

    > 保存系统上man文本的目录

19. LPDEST或PRINTER

    > 保存缺省打印机名，用于打印作业时指定打印机名。

### 导出变量以便在其他脚本进程中使用

> 在脚本中定义变量后只有export后才能在其的脚本中引用修改，就像设置了全局变量一样

### 脚本中使用命令行参数

> 运行脚本，并在脚本中获取命令行传递的参数就用$n，$0为脚本名，$1...9等为相应的参数

### 特定变量参数

* $#    传递到脚本的参数个数
* $*    以一个单字符串显示所有向脚本五福星人参数。与位置变量不同，此选项参数可超过9个
* $$    脚本运行的当前进程ID号
* $!    后台运行的最后一个进程的进程ID号
* $@    与$#相同，但是使用时加引号，并在引号中返回每个参数
* $-    显示shell使用的当前选项，与set命令功能相同
* $?    显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误


