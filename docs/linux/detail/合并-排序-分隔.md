## sort命令

### 语法

```bash
Usage: sort [OPTION]... [FILE]...
```

分类排序好所有的文件并合并输出

### sort选项

**-b, --ignore-leading-blanks**

> 忽略开头的空格

**-d, --dictionary-order**

> 按字典顺序排序，即只考虑空格及字母

**-f, --ignore-case**

> 转换小写字母到大写字母再比较

**-g, --general-numeric-sort**

> 按普通的数值排序

**-i, --ignore-nonprinting**

> 忽略不可打印字符，即只根据可打印字符排序

**-M, --month-sort**

> 根据日期中的月份排序，如jan<feb<...<dec

**-n, --numeric-sort**

> 根据字符数字值排序，如：`sort -t: -n aaa`

**-r, --reverse**

> 按照逆序排序，如`sort -t: -r aaa`

**-c, --check**

> 检查文件是否已经排序过

```bash
sort -c aaa
sort: aaa:2: disorder: aaaaaaa 5
```

**-k, --key=POS1[,POS2]**

> 根据某个域排序，或根据pos1到pos2之间的字符串作为域排序

```bash
#使用第2个域进行排序
sort -t: -k2 aaa

#先以第2域排序，再以第4域排序
sort -t: -k2 -k4 aaa
```

**-m, --merge** 

> 合并已排序好的文件，不做排序，如：`sort -m aaa bbb`

**-o, --output=FILE**

> 指定结果的输出文件

**-s, --stable** 

> 通过禁用最后手段比较稳定排序

**-S, --buffer-size=SIZE**

> 为主要排序缓存指定大小

**-t, --field-separator=SEP**

> 指定域分隔符，默认为空格

```bash
#指定：为域分隔符
sort -t: aaa
```

**-T, --temporary-directory=DIR**

> 指定临时文件存放目录

**-u, --unique**

> 去掉重复的行，即输出结果中没有重复的行

```bash
$cat aaa
hello,bill 2
aaaaaaa 5
ccccccc 3
ccccccc 3
ccccccc 3
bbbbbb 4
ccccccc 3
aaaaaaa 5
aaaaaaa 5
$sort -u aaa
aaaaaaa 5
bbbbbb 4
ccccccc 3
hello,bill 2
```


## uniq用法

uniq用来从一个文件文件中去除或禁止重复行，一般uniq假定文件已经分类排序，并且结果正确。

与sort -u不同的是，sort的唯一性选项去除所有重复的行，而uniq命令即去除持续不断重复出现的行，中间不夹杂任何其他文件。

### 语法

```bash
uniq [-c | -d | -u] [-i] [-f num] [-s chars] [input_file [output_file]]
```

### 其选项含义

* -i 不区分大小写
* -u 只显示不重复的行

    ```bash
    $uniq -u aaa
    hello,bill 2
    aaaaaaa 5
    bbbbbb 4
    ccccccc 3
    ```

* -d 只显示有重复数据行，每个重复行只显示一次

    ```bash
    $uniq -d aaa
    ccccccc 3
    aaaaaaa 5
    ```

* -c 打印每一行重复出现的次数

    ```bash
    $uniq -c aaa
    1 hello,bill 2
    1 aaaaaaa 5
    3 ccccccc 3
    1 bbbbbb 4
    1 ccccccc 3
    2 aaaaaaa 5
    ```

* -f num为数字，前num个域被忽略


## join命令

join连接合并两个已排序的文件，每个文件里都有一些元素与另一个文件相关，其中分隔域通常由空格或tab键分隔，也可以指定其他分隔域。一些系统要求join文件时其中的域要少于20个，如果大于20，则最好使用DBMS系统。

### 语法

```bash
join [-a file_number | -v file_number] [-e string] [-o list] [-t char] [-1 field] [-2 field] file1 file2
```

### 参数选项说明

**-a file_number**

> 表示同时输出哪个文件中不匹配的行

```bash
#文件aaa的内容
$cat aaa
bill aaa 111
bing bbb 222
john ccc 333
demo ddd 444
test fff 555

#文件bbb的内容
$cat bbb
aaa 111@126.com
bbb 222@126.com
ccc 333@126.com
ddd 444@126.com
eee 666@126.com

##同时输出两个文件中匹配的与不匹配的行，用-a指定
$join -a1 -a2 -1 2 -2 1 aaa bbb
aaa bill 111 111@126.com
bbb bing 222 222@126.com
ccc john 333 333@126.com
ddd demo 444 444@126.com
eee 666@126.com
test fff 555
```

**-1 field -2 field**

> 指定分别根据哪个文件的哪个域进行连接合并，默认是根据每个文件的第1个域进行连接，如果第1个域不匹配，则输出空

```bash
#根据aaa的第二个域及bbb的第一个域进行连接
$join -1 2 -2 1 aaa bbb
aaa bill 111 111@126.com
bbb bing 222 222@126.com
ccc john 333 333@126.com
ddd demo 444 444@126.com
```

**-e string**

> 用指定的字符串string代替空域输出

**-o list**

> 指定匹配行的哪个文件的哪个域输出

```bash
#指定输出aaa的第2，3域和bbb的第2域，而忽略了aaa的第1域
$join -o 1.2,1.3,2.2 -1 2 -2 1 aaa bbb
aaa 111 111@126.com
bbb 222 222@126.com
ccc 333 333@126.com
ddd 444 444@126.com
```

**-t char**

> 指定域分隔符，默认是空格或tab键，可以指定为：

**-v file_number**

> 指定只输出哪个文件的不匹配的行

```bash
#输出两个文件中不匹配的行
$join -v1 -v2 -1 2 -2 1 aaa bbb
eee 666@126.com
test fff 555
```

## cut命令

从标准输入或是文件中剪切列或域输出

### 语法

```bash
cut -b list [-n] [file ...]
cut -c list [file ...]
cut -f list [-d delim] [-s] [file ...]
```

### 选项说明举例

**-b list**

> 指定剪切的字节数

**-c list**

> 指定剪切的字符数

```bash
$cut -c 1,4 aaa
bl
bg
jn
do
tt
```

**-d delim**

> 指定域分隔符

**-f list**

> 指定输出哪些域

```bash
#以：为分隔符并输出第2域
$cut -d: -f 2 bbb
111@126.com
222@126.com
333@126.com
444@126.com
666@126.com
```

## paste命令

cut用来从文件或标准输出中抽取数据列，然后再用paste可以将这些数据粘贴起来形成相关文件。粘贴两个不同来源的数据时，首先将其排序分类，并确保两个文件行数相同。

### 语法

```bash
paste [-s] [-d list] file ...
```

### 选项说明举例

**-d list**

> 指定不同于空格或tab键的域分隔符，如用@分隔域，使用-d@

```bash
$paste -d '|'  aaa bbb
bill aaa 111 |aaa:111@126.com
bing bbb 222 |bbb:222@126.com
john ccc 333 |ccc:333@126.com
demo ddd 444 |ddd:444@126.com
test fff 555 |eee:666@126.com
``

**-s**

> 将每个文件合并成行而不是按行粘贴

```bash
$paste -s  aaa bbb
bill aaa 111    bing bbb 222    john ccc 333    demo ddd 444    test fff 555
aaa:111@126.com bbb:222@126.com ccc:333@126.com ddd:444@126.com eee:666@126.com
```

**-**

> 配合管道以几列的形式显示，一个-表示一列

```bash
#用空格分隔，以三列显示
$ls | paste -d ' ' - - -
Applications Desktop Documents
Downloads GitHub Library
Movies Music Pictures
Public aaa bbb
django get-pip.py test.php
百度云同步盘
```

## split命令

把大文件分割成几个小的文件

### 语法

```bash
split [-a suffix_length] [-b byte_count[k|m]] [-l line_count] [-p pattern] [file [name]]
```

### 选项说明及举例

**-a suffix_length**

> 指定分割的文件名前缀长度

```bash
#指定前缀长度为5
$split -l 3 -a 5 aaa
##分割后的文件
xaaab
xaaac
```

**-b byte_count[k|m]**

> 指定每多少个字节分割分一个文件

**-l line_count**

> 指定每多少行分割为一个文件

```bash
#指定每3行分割为一个文件
$split -l 3 -a 5 aaa
```

**-p pattern**

> 当文件的行匹配指定的模式时才进行分割

```bash
#当行中匹配bing时才分割
$split -a 5 -p bing aaa
```

