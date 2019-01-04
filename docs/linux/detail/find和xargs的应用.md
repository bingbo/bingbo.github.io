查找文件命令：

```bash
find pathname -options [-print -exec -ok]
```

* pathname  查找的目录路径，例如当前目前.，根目录/等
* -print 将匹配的文件输出到标准输出
* -exec 对匹配的文件执行指定的命令，形式为'command {} \;'
* -ok 和exec作用相同，只是每次执行相应的命令时都会先给出确认提示等

### options命令选项

> -name 按照文件名查找

例如查找所有的*.txt文件：

```bash
find . -name '*.txt' -print
```

> -perm 按照文件权限来查找

如查找文件权限为755的文件：

```bash
find . -perm 755 -print
```

> -prune 跳过不在指定目录中查找

该选项指出需要忽略的目录，如果同时使用了-depth选项，则-prune选项就会忽略，如下所示在/apps下查找但不查找/apps/bin目录：

```bash
find /apps -name '/apps/bin' -prune -o -print
```

> -user 按照文件拥有者查找

查找拥有者为dave的文件：

```bash
find ~ -user bingbo -print
```

> -group 按照文件组查找

查找属于act用户组的文件:

```bash
find /apps -group act -print
```

> -mtime -n +n 按照文件的更改时间查找，-n表示文件更改时间距现在n天以内，+n表示文件更改时间距现在n天以前。-atime和-ctime与此类似。

查找更改时间在5日以内的文件：

```bash
find / -mtime -5 -print
```

查找更改时间在3日以前的文件：

```bash
find /var/lib -mtime +3 -print
```

> -nogroup 查找无有效所属组的文件，即该文件所属组在/etc/groups中不存在

查找没有有效所属用户组的所有文件：

```bash
find -nogroup -print
```

> -nouser 查找无有效属主的文件，即该文件的属主在/etc/passwd中不存在

```bash
find /home -nouser -print
```

> -newer file1 ! file2 查找更改时间比文件file1新但比文件file2旧的文件

!是逻辑非符号，查找更改时间比aa.txt新但比bb.txt文件旧的文件：

```bash
find . -newer aa.txt ! -newer bb.txt -exec ls -l {} \;
```

> -type 查找某一类型的文件

文件类型有：

* b - 块设计文件
* d - 目录
* c - 字符设备文件
* p - 管道文件
* l - 符号链接文件
* f - 普通文件

查找当前目录下所有的目录：

```bash
find . -type d -print
```

> -size n[c] 查找文件长度为n块的文件，带有c时表示文件长度以字节计

按文件长度来查找文件，如查找文件长度大于1M字节的文件：

```bash
find . -size +1000000c -print
```

查找长度恰好为100字节的文件：

```bash
find . -size 100c -print
```

> -depth 在查找文件时，首先查找 当前目录中的文件，然后再在其子目录中查找 

从根目录依次查找子目录名为test.txt的文件：

```bash
find / -name 'test.txt' -depth -print
```

> -fstype 查找位于某一类型文件系统中的文件，这些文件系统类型通常可以在配置文件/etc/fstab中找到，该配置文件中包含了本系统中有关文件系统的信息


> -mount 在查找文件时不跨越文件系统mount点

在当前文件系统中查找文件（不进入其他文件系统），如从当前目录开始查找位于本文件系统中文件名以.txt结尾的文件：

```bash
find . -name '*.txt' -mount -print
```

> -follow 如果find命令遇到符号链接文件，就跟踪到链接所指向的文件

```bash
find . -name 'aa.txt' -follow
```

> -cpio 对匹配的文件使用cpio命令，将这些文件备份到磁带设备中

该选项可以用来向磁带设备备份文件或从中恢复文件。如果希望使用cpio命令备份/etc、/home和/apps目录中的文件，可以使用以下命令：

```bash
cd /
find etc home apps -depth -print | cpio -ivcdC65536 -o /dev/rmt0
```

> -exec | -ok

查找以.log结尾且更改时间在5日以上的文件，并删除他们：

```bash
find . -name '*.log' -mtime +5 -ok rm {} \;
```

### xargs

-exec选项处理匹配的文件时，find命令会把所有的文件一起传递给exec执行，而有些系统对传递给exec的长度有限制；而xargs命令每次只获取一部分文件而不是全部。同时使用-exec选项时在有些系统中会为处理每一个匹配到的文件发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行，这样在一些情况下进程过多，性能会下降，而xargs则只有一个进程。

查找文件并显示文件类型：

```bash
find / -type f -print | xargs file

#删除.svn目录文件
find ./ -type d -name .svn | xargs -i rm -rf {};
#或
find ./ -name .svn -exec rm -rf {} \;
```
