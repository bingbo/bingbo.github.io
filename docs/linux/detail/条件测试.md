## 测试语法

```bash
test condition
#或
[ condition ]
```

> 使用方括号时，要注意在条件两边加上空格。

## 文件测试

测试状态：

| 符号|释义
|----|------
|-d | 是否是目录
|-f | 是否是文件
|-L | 是否是符号连接
|-r | 是否可读
|-s | 是否非空文件长度大于0
|-w | 是否可写
|-u | 是否文件有suid位设置
|-x | 是否可执行

```bash
$test -w test.txt
$[ -w test.txt ]
$echo $?
0
```

## 逻辑操作符

|符号|释义
|---|---
|-a | 逻辑与
|-o | 逻辑或
|!  | 逻辑否

```bash
[ -w test.txt -a -x test.txt ]
[ -w test.txt -o -x test.txt ]
```

## 字符串测试

### 语法

```bash
test "string"
test "string1" operator "string2"
[ operator "string" ]
[ "string1" operator "string2" ]
```

### 字符串操作符

|符号|释义
|---|---
|= | 两字符串相等
|!=| 不相等
|-z| 空串
|-n| 非空串

```bash
[ $name = 'bill' ]
[ -z $name ]
```

## 数值测试

### 语法

```bash
test "num1" operator "num2"
[ "num1" operator "num2" ]
```

### 数值操作符

|符号|释义
|---|---
|-eq| 数值相等
|-ne| 数值不相等
|-gt| 第一个数大于第二个数
|-lt| 第一个数小于第二个数
|-le| 第一个数小于等于第二个数
|-ge| 第一个数大于等于第二个数

```bash
[ $num -eq "30" ]
[ "30" -lt "40" ]
```

## 算术操作

expr命令一般用于算术操作：

```bash
expr param1 operator param2
```

例如：

```bash
$expr 30 / 3
10

num = 1
num = `expr $num + 1`

##模式匹配
$value=accouts.doc
$expr $value : '\(.*\).doc'
accounts
```
