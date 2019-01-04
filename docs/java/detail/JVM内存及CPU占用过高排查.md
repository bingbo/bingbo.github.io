1. 通过top命令查看当前CPU及内存情况

   ```bash
   top
   86786  java         98.4 13:22.7
   ```

2. 获得pid,通过`top -H -p86786`查看有问题的线程

   > 说明： -H 指显示线程，-p 是指定进程

3. 可以看到两个CPU或内存占用较高的线程，记下PID（ 此处的PID即为线程ID标识） ，将其从十进制转成十六进制表示，如0x7f1

    ```bash
    printf "%x\n" tid
    0x7f1
    ```

4. 通过jstack命令获取当前线程栈，可暂时保存到一个文件tempfile.txt中，在tempfile.txt中查找nid=0x7f1的线程
    
    ```bash
    jstack pid | grep '0x7f1'
    ```
    

***

1. `ps -ef|grep java` 或 `ps -ef|grep java` 查进程ID
2. `ps -mp 8514 -o THREAD,tid,time,pmem` 查出进程下线程信息，可以查出%CPU高,TIME长的线程ID(tid)
3. `printf "%x\n" 8593` tid(8593)转化为十六进制2191
4. `jstack 8514|grep 2191` 打印出堆栈信息

***

> 查看指定进程下线程的ID，时间，cpu，内存情况并根据消耗时间排序

```bash
ps -mp 9793 -o user,pid,ppid,tid,time,%cpu,%mem,cmd | sort -rk5
```