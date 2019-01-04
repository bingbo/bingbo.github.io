通过使用 tr，您可以非常容易地实现 sed 的许多最基本功能。您可以将 tr 看作为 sed 的（极其）简化的变体：它可以用一个字符来替换另一个字符，或者可以完全除去一些字符。您也可以用它来除去重复字符。这就是所有 tr 所能够做的。 

tr用来从标准输入中通过替换或删除操作进行字符转换。tr主要用于删除文件中控制字符或进行字符转换。使用tr时要转换两个字符串：字符串1用于查询， 字符串2用于处理各种转换。tr刚执行时，字符串1中的字符被映射到字符串2中的字符，然后转换操作开始

```bash
tr [-Ccsu] string1 string2
tr [-Ccu] -d string1
tr [-Ccu] -s string1
tr [-Ccu] -ds string1 string2
```

* -c 用字符串1中字符集的补集进行替换
* -d 删除字符串1中所有输入字符
* -s 删除所有重复出现的字符序列

### 字符范围

* [a-z] a-z内的字符组成的字符串。
* [A-Z] A-Z内的字符组成的字符串。
* [0-9] 数字串。
* /octal 一个三位的八进制数，对应有效的 A S C I I字符。
* [O*n] 表示字符O重复出现指定次数n。因此[ O * 2 ]匹配O O的字符串

### tr中的控制字符

|速记符| 含 义| 八进制方式
|------|-----|-----
|\a |Ctrl-G 铃声| \007
|\b |Ctrl-H 退格符| \010
|\f |Ctrl-L 走行换页| \014
|\n |Ctrl-J 新行 |\012
|\r |Ctrl-M 回车 |\015
|\t |Ctrl-I tab键| \011
|\v |Ctrl-X |\030


### 去除重复的小写字符

```bash
$echo 'aabbcdef' | tr -s '[a-z]'
abcdef
```

### 删除空行，换行的八进制表示为\012

```bash
$cat test.txt
aaaa

cccc

dddd
$tr -s '[\012]' < test.txt
aaaa
cccc
dddd

$tr -s '[\n]' < test.txt
aaaa
cccc
```

### 大小写转换

```bash
#小写转大写
$echo "hello,bill" | tr '[a-z]' '[A-Z]'
HELLO,BILL
$echo "hello,bill" | tr '[:lower:]' '[:upper:]'
HELLO,BILL

#大写转小写
$echo 'HELLO,BILL' | tr '[A-Z]' '[a-z]'
hello,bill
$echo 'HELLO,BILL' | tr '[:upper:]' '[:lower:]'
hello,bill
```

### 删除指定字符

删除数字保留字符串，用-cs来实现，-c选项表明压缩所有新行，-c表明保留所有字母不动

```bash
$echo 'hello,bill 10:50' | tr -cs '[a-z][A-Z]' '[\012*]'
hello
bill
```
