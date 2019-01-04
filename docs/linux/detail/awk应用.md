awk，在通常情况下，将每个输入行解释为一条记录而将那一行上的每个单词（由空格或制表符分隔）解释为一个字段。$0代表整个输入行。$1,$2,$3……表示输入行上的各个字段.

## 语法

```bash
awk [-F field-separator] 'commands' input-file
```

> commands是真正的命令，-F指定域分隔符，默认分空格

```bash
awk -f awk-script-file input-file
```

> -f选项指明 在文件awk-script-file中的awk脚本，即通过文件了运行awk命令,input-file是输入文件


## 应用特点

### 模式与动作

**BEGIN和END模式**

> BEGIN使用在任何文本浏览动作之前，常用于设置计数和打印头；END用来在awk完成文本浏览动作后打印输出文件总数和结尾状态标志，如不特别指明，则总是匹配或打印行数

**实际动作**

> 实际动作多用来打印或处理浏览记录，要包括在{}内

如打印开头、中间、结尾并输出到控制台及文件：

```bash
awk 'BEGIN{print "begin";} {print $0;} END{print "end";}' test.txt | tee data.out
```

### 条件操作符

* < 小于
* <= 小于等于
* == 等于,精确匹配

    `awk '{if($3 == "48"){print $0;}' data.txt`

* != 不等于
* >= 大于等于
* ~ 匹配正则表达式，表达式用斜线括起来，如/green/

    `awk '{if($3 ~ /green/){print $3;}}' data.txt`

    或关系匹配：`awk '$0 ~ /(green|red)/' data.txt`

    关系条件运算符(&&、||、！)：`awk '{if($3 == "test1" && $4 == "test2"){print $0;}}' data.txt`

* !~ 不匹配正则表达式

    `awk '$0 !~ /green/' data.txt`

### awk内置变量

* ARGC  

   > 命令行参数个数

* ARGV  
   
   > 命令行参数排列，即参数数组

* ENVIRON   

   > 支持队列中系统环境变量的使用

* FILENAME  

   > awk浏览的文件名

* FNR   

   > 浏览文件的记录数，目前操作记录数。其变量值小于等于NR。如果脚本正在访问许多文件，每一新输入文件都将重新设置此变量。

* FS    

   > 设置输入域分隔符，等价于命令行-F选项

* NF    

   > 浏览记录的域个数，则$NF表示最后一个域，如：

    ```bash
    pwd
    /usr/local/etc
    #显示$PWD目录
    echo $PWD | awk -F/ '{print $NF}'
    etc
    #显示文件名
    echo "/user/local/data.txt" | awk -F/ '{print $NF}'
    data.txt
    ```
* NR    

   > 已读的记录数

* OFS   

   > 输出域分隔符，缺省为空格，如果想设置为#，写入OFS="#"

* ORS   

   > 输出记录分隔符，缺省为新行（\n）

* RS    

   > 控制记录分隔符，缺省为新行（\n）

### awk操作符

* =、+=、*=、/=、%=、^=     赋值操作符
* ？    条件表达操作符
* ||、&&、！    并、与、非
* ~、!~     匹配操作符，包括匹配和不匹配
* <、<=、==、!=、>>     关系操作符
* +、-、*、/、%、^      算术操作符
* ++、--        前缀和后缀

### awk内置函数

* cos(x) 

   > 返回x的余弦

* exp(x) 

   > 返回e的x次幂

* int(x) 

   > 返回 x的整数部分的值

* log(x) 

   > 返回 x 的自然对数(以e为底)

* sin(x)
   > 返回x的正弦

* sqrt(x)

   > 返回x的平方根

* atan2(y,x) 

   > 返回y/x的反正切，其值在－pai到－pai之间

* rand() 

   > 返回伪随机数r,其中0<=r<1

* srand(x)

   > 建立rand()的新的种子数，如果没有指定种子数，就用当天的时间。返回环境旧的种子值

* gsub(r,s,t)
   
   > 在字符串t中用字符串s替换和正则表达式r匹配的所有字符串。返回替换的个数。如果没有给出t，默认为$0

* index(s,t) 

   > 返回子串t在字符串s中的位置，如果没有指定s,则返回0

* length(s) 

   > 返回字符串s的长度，当没有给出s时，返回$0的长度

* match(s,r) 

   > 如果正则表达式r在s中出现，则返回出现的起始位置；如果在s中没有发现r，则返回0.设置RSTART和RLENGTH的值

* split(s,a,sep) 

   > 使用字段分隔符sep闺怨字符串s分解到数组a的元素中，返回元素的个数

* sprintf() 

   > 格式化输出

* sub(r,s,t) 

   > 在字符串t中用s替换正则表达式r的首次匹配。如果成功则返回1，否则返回0，如果没有给出t，默认为$0

* substr(s,p,n) 

   > 返回字符串s中从位置p开始最大长度为n的子串。如果没有给出n,返回从p开始剩余的字符串

* tolower(s) 

   > 将字符串s中的所有大写字符转换成小写，并返回 新串

* toupper(s) 

   > 将字符串中s中的小写字符转换成大写，并返回新串

### awk字符串转义序列

* \b    退格键
* \t    tab键
* \f    换页
* \ddd  八进制值
* \n    新行
* \c    任意其他特殊字符，如\\为反斜线符号
* \r    回车键

### 格式化输出printf

**printf修饰符**

* -     左对齐,`awk '{printf "%-15 %s\n", $1,$3}' data.txt`
* width     域的步长，用0表示0步长
* .prec     最大字符串长度，或小数点右边的位数

**printf格式**

* %c    ASCII字符
* %d    整数
* %e    浮点数，科学记数法
* %f    浮点数，如123.44
* %g    awk决定使用哪种浮点数转换e或f
* %o    八进制数
* %s    字符串
* %x    十六进制数

### awk命令传参数传值

格式：`awk var=val '{print var}' data.txt`

例如：`awk age = 10 '{if($5 < age){print $0}}' data.txt`

### awk脚本文件

一些比较多的命令脚本可以写到文件中再执行，这样不必每次都在命令行输入，且可以增加注释，同时脚本文件名最好以.awk命名较好，如下所示：

```bash
awk '
    BEGIN { print "this is the awk begin.......";
            blankNum=0;
    }
    {
        if(/^$/) { print "this is a blank line";print ++blankNum; }
        #w使用~操作符可以测试一个字段的正则表达式：
        if($5 ~ /MA/) { print $5; }
        #可以使用!~来反转这个规则的意义
        if($5 !~ /MA/) {
            print $5;
        }else if(1<2){
            print "1<2";
        }else{}

        #条件操作符
        print (50>40)?"50>40":"50<40";

        #while循环 while(condition) action
        while(i<5){
            i++;
            print "blankNum < 10";
        }
        #do循环 do action while(condition)
        do{
            j++
            print "blankNum < 10";
        }while(j<5)
        #for循环for(set_counter;test_counter;increment_counter) action
        for(i=0;i<4;i++){
            print "this is for test";
        } 
    }
    END{}
    ' data.txt
```

### 数组

```bash
array[subscript]=value
```

**关联数组**

即数组下标可以不是数字，如：`name['bill']='bill'`

**数组循环**

```bash
#for(varialbe in array){
# do something with array[variable]
# }
arr[0]=1;
arr[2]=3;
arr['name']="bill";
arr['email']="bill\@126.com";
for(item in arr){
    print item , arr[item];
}
```

**用split()创建数组**

```bash
n=split(string,array,separator)
```

例如：

```bash
z=split("zhang bing bing",array," ");
for(i=1;i<=z;i++){
    print i,array[i];
}

**删除数组元素**

`delete array[subscript]`

**多维数组array[2,4]**

**访问环境变量**

```bash
for(env in ENVIRON){
    print ENVIRON[env];
}
```


## 注意事项

* 错误避免

    * 确保整体awk命令用单引号括起来
    * 确保命令内所有引号成对出现
    * 确保用花括号括起动作语句，用圆括号括起条件语句
    * 可能忘记使用花括号，也许你认为没有必要，但awk则会解释出错

* 设置输入域到变量名

    给每个域设置一个可读的变量，更有助于编程，如：

    ```bash
    awk '{name = $1; password = $2; email = $3; if(name == "bill") {print name;}}' data.txt
    ```

* 用引号将数字引用起来是可选的，即"27",27产生同样的结果
* 修改域值，如：`$1=$1+1`
* 创建新域，如只有3个域，新加域：`$4 = $2 + $3`

## 部分样例

### 日志统计

```bash
#!/bin/bash  

function get_pv(){
 date=$1
 awk -v date=$date '
  BEGIN{
   index_pv=0;
   search_pv=0;
   city_pv=0;
  }
  {
   if($0 ~ /ftype/){
    search_pv++;
   }
   if($0 ~ /index=1/){
    index_pv++;
   }
   if($0 ~ /index=1&superphone_city=1/ && $0 !~ /index=1&query_city_name/){
    city_pv++;
   }
  }
  END{
   print date,"total_index:",index_pv,"index_pv:",index_pv-search_pv-city_pv,"search_pv:",search_pv,"city_pv:",city_pv;
  }
 ' access.$date > phone_pv.$date
}

function get_uv(){
 date=$1
 awk '
  BEGIN{}
  {
   if($0 ~ /BAIDUID/){
    pos=index($0,"BAIDUID");
    baiduid=substr($0,pos,length("BAIDUID=D2F04E3489B46E6A5844471D75780220:FG=1"));
    print baiduid;
   }
  }
  END{}
 ' access.$date|sort|uniq > phone_uv_tmp.$date
 #awk -v date=$date'
 #	BEGIN{
 #	count=0;
 #	}
 #	{
 #	count++;
 #	}
 #	END{
 #	print date,count;
 #	}
 #' phone_uv_tmp.$date > phone_uv.$date
}

startdate=20140919
days=7
for((i=0;i<$days;i++))
do
 dates[$i]=`date -d "+$i day $startdate" +"%Y%m%d"`
    echo ${dates[$i]}
 get_pv ${dates[$i]}
 get_uv ${dates[$i]}
done
```

### IP计数排序

```bash
cat 1119IP.txt.bak | awk 'BEGIN{arr[0]=1;}{if(arr[$0]){arr[$0]++;}else{arr[$0]=1;}}END{for(item in arr){print item,arr[item]}}' | sort -n -r -k2 >a.txt

cat error_log.2014122221 | awk 'BEGIN{}{pos=index($0,", logid:");pos1=index($0,"client:");print substr($0,pos1+7,pos-pos1-7)}END{}' > IP.txt

cat error_log.2014122221 | awk 'BEGIN{}{if($0 ~ /21:07:00/){print $0;}}END{}' > error.log_210700
cat tmp/jumps.b2log.2015011304 | awk '{pos=index($0,"statistics");pos1=index($0,"custom_param");tj=substr($0,pos+11,pos1-pos-11);if(tj !~ /unset/){print tj;}}'
```

### 根据同一字段合并两文件

```bash
awk 'NR==FNR{a[$2]=$1}NR>FNR{print $0,a[$2]}' a1.txt a2.txt 

a1.txt:
a aaa
b bbb
c ccc

a2.txt:
111 bbb
333 ccc
444 aaa
```
