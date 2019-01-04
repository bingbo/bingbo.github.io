xhprof是php性能优化工具，可以显示程序运行过程中的一些函数调用和执行过程所消耗的时间、内存等信息，且可以以图表的形式展现程序流过程中的性能信息

## 安装php

> 注：这里目前必须是php5版本的，php7目前还不能与xhprof很好的结合使用，会有编译错误发生

具体安装详见：[PHP扩展](https://github.com/bingbo/blog/wiki/PHP%E6%89%A9%E5%B1%95%E5%BC%80%E5%8F%91)

## 编译安装xhprof

### 下载并编译

```bash
##下载安装xhprof
wget http://pecl.php.net/get/xhprof-0.9.3.tgz
tar zxvf xhprof-0.9.2.tgz
cd xhprof-0.9.2/extension/
/home/work/php5/bin/phpize
./configure --with-php-config=/home/work/php5/bin/php-config
make
```

### 配置php添加xhprof扩展

将上一步编译生成的扩展库xhprof.so文件cp到php扩展库目录下，可`php --ini`查看配置文件中的extention_dir目录

修改php.ini配置

```php
[xhprof]
extension=xhprof.so
;下面的这个地址是用来保存测量记录的目录，在页面输出测量得到的数据的时候，它会自动到这儿来找输出的文件。
xhprof.output_dir=/home/work/tmp
```

重启php-fpm

### 为入口文件引入xhprof

```php
//开启调试
xhprof_enable();
// cpu:XHPROF_FLAGS_CPU 内存:XHPROF_FLAGS_MEMORY
// 如果两个一起：XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY 
//xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);
xhprof_enable(XHPROF_FLAGS_NO_BUILTINS | XHPROF_FLAGS_CPU | XHPROF_FLAGS_MEMORY);

//要测试的php代码
foo();

//停止监测
$xhprof_data = xhprof_disable();

// display raw xhprof data for the profiler run
print_r($xhprof_data);

//包含工具类，在下载的 tgz 包中可以找到
$XHPROF_ROOT = realpath(dirname(__FILE__) .'/..');
include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_lib.php";
include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_runs.php";

// save raw data for this profiler run using default
// implementation of iXHProfRuns. 
$xhprof_runs = new XHProfRuns_Default();

// xhprof_foo 指命名空间，可以为任意字符串
$run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_foo");
//这里是可视化界面访问地址
\Yii::info('http://xxx.xxx.xxx:8101/xhprof_html/index.php?run=57b17ee196790&source=xhprof');
```

前提是必须先把xhprof_lib和xhprof_html相关目录copy到可访问的根目录下，然后用http://xxx.xxx.xxx:8101/xhprof_html/index.php?run=57b17ee196790&source=xhprof即可访问可视化界面

### 数据说明

```php
Array
(
    [foo==>bar] => Array
        (
            [ct] => 2        # number of calls to bar() from foo()
            [wt] => 37       # time in bar() when called from foo()
            [cpu] => 0       # cpu time in bar() when called from foo()
            [mu] => 2208     # change in PHP memory usage in bar() when called from foo()
            [pmu] => 0       # change in PHP peak memory usage in bar() when called from foo()
        )
.....
```

在查看 Xhprof 或 XHProf UI 展示的性能数据时，会遇到以下几个术语，其含义对应如下：

* Calls / Call Count：函数/方法被调用的次数
* Incl. Wall Time / Wall Time：执行该函数/方法实际耗费的时间
* Incl. MemUse / Memory Usage：该函数/方法当前占用的内存
* Incl. PeakMemUse / Peak Memory Usage：函数/方法占用内存的峰值（注：我也不知道这个峰值是怎么算的）
* Incl. CPU / CPU：执行该函数/方法，花费的CPU时间
* Excl. Wall Time / Exclusive Wall Time
* Excl. MemUse / Exclusive Memory Usage
* Excl. PeakMemUse / Exclusive Peak Memory Usage
* Exclusive CPU

> 术语中的Incl表示Inclusive，Excl表示Exclusive。Inclusive表示测量到的数据是函数/方法本身及所有调用的子函数/方法总共耗费占用的资源，Exclusive则表示不包含调用的子函数/方法耗费占用的资源。另外，所有测量值都是每个函数/方法调用在次数上的叠加。

__Tips：__

> 点击[View Full Callgraph]查看图片的时候报错：failed to execute cmd：" dot -Tpng". stderr：`sh： dot：command not found`。
原因：原因：未安装图形化工具Graphviz

```bash
#centos ：
yum install graphviz
#unbutu:
apt-get install graphviz
```


> 在yii、thinkphp中使用xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);报502错误。应该修改为以下配置。

```php
xhprof_enable(XHPROF_FLAGS_NO_BUILTINS | XHPROF_FLAGS_CPU | XHPROF_FLAGS_MEMORY);
```