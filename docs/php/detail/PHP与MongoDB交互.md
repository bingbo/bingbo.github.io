## 安装mongodb驱动

想要php与mongodb结合使用，就得先装mongodb驱动，即安装php的mongodb扩展

### mac上mongodb驱动安装

1. 查看php相应版本

   ```shell
   $php -v
   PHP 5.5.32 (cli) (built: Feb  5 2016 14:01:05)
   Copyright (c) 1997-2015 The PHP Group
   Zend Engine v2.5.0, Copyright (c) 1998-2015 Zend Technologies
   ```

2. 用brew查找并安装相应版本驱动

   ```shell
   $brew search mongodb
   homebrew/php/php54-mongodb         homebrew/php/php55-mongodb ✔       
   homebrew/php/php56-mongodb         homebrew/php/php70-mongodb         
   mongodb ✔
   ```

   安装：`$brew install homebrew/php/php55-mongodb`
   
### 配置添加php扩展

   由第2步已经生成了相应的mongodb.so文件及ext-mongodb.ini配置文件，这里需要把其配置文件copy到php额外的配置文件目录中，然后重启php，如：

   ```shell
   $php --ini
   Scan this dir for additional .ini files	/Library/Server/Web/Config/php   ##扩展配置文件目录
   $cp /usr/local/etc/php/5.5/conf.d/ext-mongodb.ini /Library/Server/Web/Config/php/
   $php-fpm -c /usr/local/etc/php/5.5/php.ini -y /usr/local/etc/php/5.5/php-fpm.conf
   ```

## 使用PHP Library与mongodb交互

### 安装PHP Library

   ```shell
   $ composer require "mongodb/mongodb=^1.0.0"
   ```

   会生成composer.json, composer.lock等文件和vendor目录，目录中包含了mongodb相关的类及源码。

### 使用PHP Library

   ```php
   <?php
   require 'vendor/autoload.php';

   $c = new MongoDB\CLient('mongodb://localhost:27017');
   $user = $c->test->user;
   //$result = $user->insertOne(['name'=>'bing','email'=>'bing@126.com','age'=>30]);
   //echo $result->getInsertedId();

   $all = $user->find();
   foreach($all as $key=>$value){
      echo $value->id.'|'.$value->name.'|'.$value->email.'|'.$value->age;
      echo "</br>";
   }
   ?>
   ```
   
### PHP Library中有关mongodb的主要类

主要是`Client`、`DataBase`和`Collection`等类，以及该三个主要类中所涉及包含的操作方法类等，如下图所示：

![](https://github.com/bingbo/blog/blob/master/images/phpmongodb%E5%BA%93%E4%B8%BB%E8%A6%81%E7%B1%BB%E5%9B%BE.jpg)
### 其他安装使用方法
安装参考：[其他平台及方法安装](http://docs.php.net/manual/en/mongodb.setup.php)