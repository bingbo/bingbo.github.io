## 函数

### 语法

```bash
fname ()
{
    command1
    ...
}

#或加function
function fname()
{
    command1
    ...
}
```

### 示例

```bash
##定义函数
hello(){
    echo "say hello to $1"
    return
}
##使用函数并传参数
hello bill
```

**返回值由return指定**

* return    从函数中返回，用最后状态命令决定返回值
* return 0  无错误返回
* return 1  有错误返回

> 通过`var=hello`可以获得返回结果

**查看删除函数**

`. /fun.sh`可载入shell

一旦文件载入shell，就可以在命令行或脚本中调用函数，可以用set命令查看所有定义的函数。

用unset可以删除函数。

## 参数操作shift和getopts

### shift操作

一般的unix命令均有`命令 选项 文件`等的形式，而选项部分最多可包含12个不同的值，而shift可以去除只使用$1到$9传递参数的限制。

向脚本传递参数时，有时需要将每个参数偏移以处理选项，这就是shift的功能，它每次会将参数位置向左偏移一位。

```bash
##查看所有参数
while [ $# -ne 0 ]
do
    echo $1
    shift
done

##查看命令行中最后一个参数
eval echo \$$#
```

### getopts操作

getopts用于形成命令行处理标准形式，原则上讲，脚本应具有确认带有多个TF爱的人命令文件标准格式的能力，即带什么样的参数已经定好了

一般格式为：`getopts optionstr var`

输入的选项与optionstr进行匹配，如果匹配，则var就赋相应的参数值，如果未发现匹配字符，则变量var为？。getopts接收完所有参数后，返回非零状态，意即参数传递成功。

```bash
getopts ahfvc: option
```

上面说明，有a、h、f、v、c等选项，而选项c必须指定相应的值，而`OPTARG`变量用来保存其选项值

```bash

while getopts :brsqc: option
do
    case $option in
        b)
            echo "b means begin"
            ;;
        r)
            echo "r means reload"
            ;;
        s)
            echo "s means stop"
            ;;
        q)
            echo "q means quit"
            ;;
        c)
            var=$OPTARG
            echo "c must get value, and this value is $var"
            ;;
        \?)
            echo "`basename $0` -[b r s q] -[c value]"
            ;;
    esac
done
```

> _这里:brsqc:的前面的冒号表示如果c不传值的话也不会出错误，即会屏蔽错误信息提示，没有前置冒号的话会有错误信息。_ 

在脚本中指定命令行选项时，其命名规则最好与unix一致，如下所示：

|选项|含义
|----|----
|-a | 扩展
|-c | 计数、拷贝
|-d | 目录、设备
|-e | 执行
|-f | 文件名、强制
|-h | 帮助
|-i | 忽略状态
|-l | 注册文件
|-o | 完整输出
|-q | 退出
|-p | 路径
|-v | 显示方式或版本