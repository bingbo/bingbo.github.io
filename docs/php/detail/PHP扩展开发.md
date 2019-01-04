
## 扩展开发

### 新建扩展模块

```php
./ext_skel --extname=test
#该命令生成模块test相应的目录及文件
#去掉引入扩展模块方法注释，编译php时可加--with-test方式引入
vim config.m4
#修改如下内容
PHP_ARG_WITH(test, for test support,
dnl Make sure that the comment is aligned:
[  --with-test             Include test support])
```

### 编写扩展函数

> 编写修改test.c文件，添加tmax函数

```c
/*修改添加tmax函数实现获取两个数中的最大数*/
PHP_FUNCTION(tmax)
{
    long a, b, res;
    size_t arg_len;
    zend_string *strg;

    if (zend_parse_parameters(ZEND_NUM_ARGS(), "ll", &a, &b, &arg_len) == FAILURE) {
        return;
    }

    if(a > b){
        res = a;
    }else{
        res = b;
    }

    RETURN_LONG(res);

}
```

### 编写扩展类

> 添加扩展类User及其方法

```c
zend_class_entry *user_ce;
/*定义getName方法*/
ZEND_METHOD(User, getName){
    printf("this is the public method: getName\n");
}

/*定义getEmail方法*/
ZEND_METHOD(User, getEmail){
    printf("this is the public method: getEmail\n");
}

/*定义构造函数方法*/
ZEND_METHOD(User, __construct){
    printf("this is the construct method\n");
}

/*指定添加的方法及访问类型*/
static zend_function_entry user_method[] = {
    ZEND_ME(User, getName, NULL, ZEND_ACC_PUBLIC)
    ZEND_ME(User, getEmail, NULL, ZEND_ACC_PUBLIC)
    ZEND_ME(User, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
    {NULL, NULL, NULL}
};
```

> 注册相应的类及方法属性

```c
/* {{{ PHP_MINIT_FUNCTION
 */
PHP_MINIT_FUNCTION(test)
{
	/* If you have INI entries, uncomment these lines
	REGISTER_INI_ENTRIES();
	*/
    zend_class_entry uce;
    INIT_CLASS_ENTRY(uce, "User", user_method);
    user_ce = zend_register_internal_class(&uce TSRMLS_CC);

    //定义公有变量version并赋值为1.11
    zend_declare_property_double(user_ce, "version", strlen("version"), 1.11, ZEND_ACC_PUBLIC TSRMLS_CC);
    //定义类常量AUTHOR并赋值为bill
    zend_declare_class_constant_string(user_ce, "AUTHOR", strlen("AUTHOR"), "bill");
	return SUCCESS;
}
/* }}} */
```

### 编写扩展接口

> 添加扩展接口People及其方法getPassword

```c
zend_class_entry *people_ce;
/*未实现的方法getPassword*/
static zend_function_entry people_method[] = {
    ZEND_ABSTRACT_ME(people_ce, getPassword, NULL)
    {NULL, NULL, NULL}
};
```

> 注册接口People及其方法

```c
/* {{{ PHP_MINIT_FUNCTION
 */
PHP_MINIT_FUNCTION(test)
{
	/* If you have INI entries, uncomment these lines
	REGISTER_INI_ENTRIES();
	*/
    zend_class_entry pce;

    INIT_CLASS_ENTRY(pce, "People", people_method);
    people_ce = zend_register_internal_interface(&pce TSRMLS_CC);

	return SUCCESS;
}
/* }}} */
```

### 编写扩展抽象类

> 添加扩展抽象类Person及方法

```c
zend_class_entry *person_ce;

/*已实现的方法*/
ZEND_METHOD(Person, sayHi){
    printf("this is the abstract Person class's sayHi method\n");
}

/*父类构造方法*/
ZEND_METHOD(Person, __construct){
    printf("this is the abstruct Person class's construct method\n");
}

/*静态方法*/
ZEND_METHOD(Person, testStatic){
    printf("this is the abstruct Person class's static method\n");
}

/*方法列表及类型*/
static zend_function_entry person_method[] = {
    /*抽象方法*/
    ZEND_ABSTRACT_ME(person_ce, sayHello, NULL)
    ZEND_ME(Person, sayHi, NULL, ZEND_ACC_PUBLIC)
    ZEND_ME(Person, testStatic, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_STATIC)
    ZEND_ME(Person, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
    {NULL, NULL, NULL}
};
```

> 注册抽象类Person及方法

```c
/* {{{ PHP_MINIT_FUNCTION
 */
PHP_MINIT_FUNCTION(test)
{
	/* If you have INI entries, uncomment these lines
	REGISTER_INI_ENTRIES();
	*/
    zend_class_entry bce;
    
    INIT_CLASS_ENTRY(bce, "Person", person_method);
    person_ce = zend_register_internal_class(&bce TSRMLS_CC);


    //定义静态属性staticvar
    zend_declare_property_string(person_ce, "staticvar", strlen("staticvar"), "test static var", ZEND_ACC_PUBLIC|ZEND_ACC_STATIC TSRMLS_CC);
	return SUCCESS;
}
/* }}} */
```

### 编译安装扩展

```bash
#清除make
$make clean
#转到扩展test目录下
$pwd
/Users/work/Public/php-src/ext/test
$php7ize
$./configure --with-php-config=/Users/work/php7/bin/php-config
#安装到php相应的扩展so目录下
$make && make install
#配置php.ini添加扩展
$vi /User/work/php7/etc/php.ini
#添加如下内容
[test]
extension=test.so

##查询扩展
$php7 -m
standard
test
tipi_demo01
tokenizer
```

## 调试扩展

需要重新以--enable-debug模式编译php，编译完后php扩展目录会变成，与之前的no-debug-non-zts-20121212路径不一致，如extensions/debug-non-zts-20121212

接着修改php-src/ext/myextension/config.m4，在最后添加上：

```c
if test -z "$PHP_DEBUG"; then
        AC_ARG_ENABLE(debug,
                [--enable-debug  compile with debugging system],
                [PHP_DEBUG=$enableval], [PHP_DEBUG=no]
        )
fi
```

然后重新以–enable-debug编译扩展

```c

~/Work/php7/bin/phpize
./configure --with-php-config=~/Work/php7/bin/php-config --enable-debug
make && make install
```

调试

```bash
##查看扩展中所有的函数
nm modules/test.so

#加载php解释器和.so到gdb
sudo gdb /Users/work/php7/bin/php

#设置断点(提示未定义过的话，仍然设置)
(gdb) break zif_confirm_myextension_compiled

#运行test.php脚本，到断点处会暂停
(gdb) run test.php

#执行下一行
(gdb) next

#打印len变量
(gdb) print len

#继续执行直到完毕
(gdb) continue

#退出gdb
(gdb) quit

#从main函数调试
(gdb) start
```

多线程调试php-fpm

首先得把php-fpm启动起来，同时记录php-fpm的pid

```
gdb
(gdb) attach xxxxx --- xxxxx为利用ps命令获得的子进程php-fpm的process id
(gdb) stop --- 这点很重要，你需要先暂停那个子进程，然后设置一些断点和一些Watch
(gdb) break 37 -- 在result = wib(value, div);这行设置一个断点,可以使用list命令察看源代码
Breakpoint 1 at 0x10808: file eg1.c, line 37.
(gdb) continue
```

## 测试扩展

> 编写test.php测试脚本

```php
<?php
//调用tmax方法比较大小
echo "3 and 9 the max is ". tmax(3,9) . "\n";
//新建User实例对象
$user = new User("bill", "bill@126.com");
//调用user对象的getName方法
echo $user->getName();
//调用user对象的getEmail方法
echo $user->getEmail();
echo "version is ".$user->version."\n";
//调用User类常量
echo "User::AUTHOR is ". User::AUTHOR ."\n";

//新建Student类并实现People接口及其getPassword方法
class Student implements People{
    
    public function getPassword(){
        echo "this is implements people's method getPassword\n";
    }
}

//新建student实例并调用getPassword方法
$s = new Student();
$s->getPassword();

//继承抽象类Person
class Man extends Person{
    //实现抽象方法
    public function sayHello(){
        echo "this is sayHello method\n";
    }
    //在静态方法里调用静态属性$staticvar
    public static function getStaticVar(){
        echo "the  static var value is ".self::$staticvar."\n";
    }
}
$m = new Man();
//调用实现方法
$m->sayHello();
$m->sayHi();
//调用静态方法
Person::testStatic();
Man::testStatic();
//调用静态属性
Man::getStaticVar();
```

> 运行脚本，查看结果

```bash
$php7 test.php
3 and 9 the max is 9
this is the construct method
this is the public method: getName
this is the public method: getEmail
version is 1.11
User::AUTHOR is bill
this is implements people's method getPassword
this is the abstruct Person class's construct method
this is sayHello method
this is the abstract Person class's sayHi method
this is the abstruct Person class's static method
this is the abstruct Person class's static method
the  static var value is test static var
```

> 如果不实现People接口的getPassword方法，结果会报如下错误

```php
//新建Student类并实现People接口及不实现其getPassword方法
class Student implements People{
    
    /*
    public function getPassword(){
        echo "this is implements people's method getPassword\n";
    }
    */
}

//新建student实例
$s = new Student();
//$s->getPassword();
```

> 错误如下

```bash
$php7 test.php
3 and 9 the max is 9
this is the construct method
this is the public method: getName
this is the public method: getEmail
PHP Fatal error:  Class Student contains 1 abstract method and must therefore be declared abstract or implement the remaining methods (People::getPassword) in /Users/work/test.php on line 8
```

> 如果没有实现抽象类Person的抽象方法sayHello，则报如下错误：

```bash
$php7 test.php
PHP Fatal error:  Class Man contains 1 abstract method and must therefore be declared abstract or implement the remaining methods (Person::sayHello) in /Users/zhangbingbing/test.php on line 31
```
