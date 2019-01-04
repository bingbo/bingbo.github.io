## sed语法

```bash
sed [-nefr] [动作]
```

### 参数

* -n    
    
    > 使用安静模式，在一般sed的用法中，所有来自STDIN的数据一般都会被列出到屏幕上，但如果加上-n参数，则只有经过sed特殊处理的那一行或操作的才会被列出来

* -e

    > 直接在命令行模式上进行sed的动作编辑

* -f 

    > sed的动作是写在一个文件中的，-f filename则指定执行相应文件中的sed脚本动作

* -r 

    > sed的动作支持的是扩展型的正则表达式的语法（默认是基础正则表达式语法）

* -i

    > 直接修改读取文件的内容，而不是由屏幕输出

### 动作说明

```bash
[addr1[,addr2]]function
```

> addr1,addr2   不见得会存在，一般代表选择进行动作的行数，即如果我要在10到20行之间进行，则"10,20[动作行为]"

**function相关参数**

* a：新增，

    > a的后面可以接字符串，而这些字符串会在新的一行出现（目前的下一行）

* c：替换，

    >c的后面可以接字符串，这些字符串可以替换addr1,addr2之间的行
    
* d：删除，

    > 因为是删除，所以d后面通常不接任何参数

* i：插入，

    > i的后面可以接字符串，而这些字符串会在新的一行出现（目前的上一行）

* p：打印，

    > 也就是将某个选择的数据打印出来，通常p会与参数sed -n 一起运行

* s：替换，

    > 可以直接进行替换的工作，通常这个s的动作可以搭配正则表达式，如`sed '1,20s/old/new/g' txt`

* = 显示文件行号
* q 匹配到第一个项后就退出
* l 显示其中与ASCII代码等的控制字符，即显示特殊字符


## 应用操作

### 以行为单位新增和删除功能

```bash
#删除2-5行
sed '2,5d' data.txt

#只删除第2行
sed '2d' data.txt

#删除第2行到最后一行，$表示最后
sed '2,$d' data.txt

#删除匹配的行
sed -i '/^a.*/d' tmp.txt

#在第2行后面添加'hello,world'
sed '2a hello,world' data.txt

#在第2行前面增加'add before'
sed '2i add before' data.txt

#在第2行后面增加2行'hello'和'world'，则每行以\结尾
sed '2a hello \
world' data.txt
```

### 以行为单位的替换与显示功能

```bash
#替换2到5行的内容为'hello,bill'
sed '2,5c hello,bill' data.txt

#仅列出文件的5-7行
cat /etc/passwd | sed -n '5,7p'
```

### 查找并替换功能

```bash
#查找第3行到最后的'bill'并替换为'bing'
sed '3,$s/bill/bing/g' data.txt

#查找bill并全部替换
sed 's/bill/bing/g' data.txt
```

### 直接修改文件内容

```bash
#将每行行尾的.换成!
sed -i 's/\.$/\!/g' data.txt

#在最后一行加入hello,world
sed -i '$a hello,world' data.txt

#在mac下如下修改：-i后不加任何名字扩展，即以原文件修改保存
sed -i '' -e 's/Bdoor/Business/g' *
```

### 打印行号及多个操作

```bash
sed -e '/bill/=' data.txt

#多操作一起
sed -e '/bill/p' -e '/bill/=' data.txt
```

### 其他

```bash
#首次匹配后退出
sed '/bill/q' data.txt
#显示文件中的所有内容包括特殊字符，后跟l
sed '1,$l' data.txt
```

