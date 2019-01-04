这里主要列出了一些常用的内嵌命令

* pwd
    
    > 显示当前目录

* set

    > 除了查看调试脚本，打开关闭shell选项，还可以在脚本内部定义变量，如：

    ```bash
    #! /bin/bash
    set bill 30
    echo $1 #输出bill
    echo $2 #输出30
    ```

* times

    > 此命令给出用户脚本或任何系统命令的运行时间。

    ```bash
    $times
    ##shell消耗时间
    0m0.128s 0m0.232s
    ##运行命令消耗的时间
    0m11.251s 0m14.772s
    ```

* type

    > 使用type命令可以查询命令是否仍驻留系统及命令的类型，如

    ```bash
    $type hello
    -bash: type: hello: not found
    $type times
    times is a shell builtin
    ```

* ulimit

    > 显示shell的运行限制选项

    * -a    显示当前限制
    * -c    显示内核垃圾大小
    * -f    限制运行进程文本v烦人输出文件的大小

    ```bash
    $ulimit -a
    core file size          (blocks, -c) 0
    data seg size           (kbytes, -d) unlimited
    file size               (blocks, -f) unlimited
    max locked memory       (kbytes, -l) unlimited
    max memory size         (kbytes, -m) unlimited
    open files                      (-n) 2560
    pipe size            (512 bytes, -p) 1
    stack size              (kbytes, -s) 8192
    cpu time               (seconds, -t) unlimited
    max user processes              (-u) 709
    virtual memory          (kbytes, -v) unlimited
    ```

* wait

    > 等待直到一个用户子进程完成，可以指定进程ID号，如未指定，则等待所有子进程完成。
