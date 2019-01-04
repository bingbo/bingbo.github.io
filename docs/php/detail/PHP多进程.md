pcntl_XXX只能运行在php CLI（命令行）环境下，在web服务器环境下，会出现无法预期的结果，请慎用！

### 在命令行下开启多进程

```php
<?php
//确保这个函数只能运行在shell中
if(substr(php_sapi_name(), 0, 3) !== 'cli'){
    die('this program can only be run in CLI mode');
}

//关闭最大执行时间限制，在cli模式下，这个语句其实不必要
set_time_limit(0);

$pid = posix_getpid();//取得主里程ID
$user = posix_getlogin();//取得用户名

echo <<<EOD
USAGE: php test.php
input strings by splitted with space to display by fork a new process

    shell executor version 1.0.0 by ibingbo

EOD;

while(true){
    $prompt = "\n{$user}$ ";
    $input = readline($prompt);

    readline_add_history($input);
    if($input == 'quit'){
        break;
    }

    $arr = explode(' ', $input);
    foreach($arr as $key => $value){
        //--------子进程开始-----------------------------------------
        $pid = pcntl_fork();//创建子进程
        if($pid == 0){//子进程
            $pid = posix_getpid();
            echo "* process {$pid} was created, and executed:\n\n";
            //eval($input);//解析命令
            echo "{$value}\n";
            exit;
        }else{//主进程
            $pid = pcntl_wait($status, WUNTRACED);//取得子进程结束状态
            if(pcntl_wifexited($status)){
                echo "\n\n* sub process: {$pid} exited with {$status}\n";
            }
        }
        //--------子进程结束-----------------------------------------
    }

}
```

### 开启一定数量的子进程执行程序逻辑

```php

<?php
defined('MAX') or define('MAX', 10);
$pnum = 0;
//确保这个函数只能运行在shell中
if(substr(php_sapi_name(), 0, 3) !== 'cli'){
    die('this program can only be run in CLI mode');
}

//关闭最大执行时间限制，在cli模式下，这个语句其实不必要
set_time_limit(0);

$pid = posix_getpid();//取得主里程ID
$user = posix_getlogin();//取得用户名

echo <<<EOD

this is a test,the user is {$user},and main program is {$pid}


EOD;



while(true){

    //如果已达到最大进程数则等待子进程结束
    if($pnum > MAX){
        $cpid = pcntl_wait($status);//等待取得子进程结束状态
        if($cpid <= 0){
            exit;
        }
        //子进程正常结束则计数减1
        if(pcntl_wifexited($status)){
            echo "the sub process {$cpid} is exited with {$status}\n";
            $pnum--;
        }
    }

    //--------子进程开始-----------------------------------------
    $pid = pcntl_fork();//创建子进程
    if($pid == 0){//子进程
        $pid = posix_getpid();
        echo "* process {$pid} was created, and executed:\n\n";
        sleep(2);
        exit;
    }else if($pid > 0){//主进程
        $pnum++;
    }else{
        echo "fail to create sub process\n";
    }
    //--------子进程结束-----------------------------------------

}
```
