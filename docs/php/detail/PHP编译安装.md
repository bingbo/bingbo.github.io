```bash
#下载源码
git clone https://github.com/php/php-src`
#安装php
./buldconf
./configure --prefix=/Users/zhangbingbing/Work/php7 --with-config-file-path=/Users/zhangbingbing/Work/php7/etc --enable-fpm --with-fpm-user=www --enable-debug --enable-mbstring --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-gd
##或是
#./configure --prefix=/home/bingbing.zhangbb/php7 --with-config-file-path=/home/bingbing.zhangbb/php7/etc --with-config-file-scan-dir=/Users/zhangbingbing/Work/php7/etc/conf.d --enable-fpm --with-fpm-user=www --enable-debug --enable-mbstring --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-gd --with-openssl --with-pcre-regex --with-zlib --enable-calendar --with-curl --enable-pcntl --enable-soap --enable-zip

make && make install
```

___问题参考___

_about gd_

```
#要先装libpng 和 libjpeg
brew install libpng
brew install libjpeg
```

_about bison_

```
brew versions bison
  3.0      git checkout b744b43 Library/Formula/bison.rb
  2.7.1    git checkout 804bcf6 Library/Formula/bison.rb
  2.7      git checkout adf87c6 Library/Formula/bison.rb

git checkout adf87c6 /usr/local/Library/Formula/bison.rb
brew install bison
brew switch bison 2.7
brew link bison --force
sudo mv /usr/bin/bison /usr/bin/bison.orig
sudo ln -s /usr/local/bin/bison /usr/bin/bison
```

_about re2c_

```
brew install re2c
```

_about openssl_

```
brew install openssl
```

_about autoconf_

```
brew install autoconf
```

> 为了不与自带或已有php冲突，为php7建立软连接

```bash
#在当前用户home下建立bin目录
mkdir bin
cd bin
ln -s /User/work/php7/bin/php php7
ln -s /User/work/php7/bin/php-config php7-config
ln -s /User/work/php7/bin/phpize php7ize
ln -s /User/work/php7/sbin/php-fpm php7-fpm
```

> 在当前用户下为php7添加全局命令

```bash
vim .bash_profile
#添加如下内容，保存并退出
PATH=$PATH:$HOME/bin
export PATH
source .bash_profile
```

> 测试成功安装

```bash
$php7 -v
PHP 7.1.0-dev (cli) (built: Mar 26 2016 09:28:49) ( NTS DEBUG )
Copyright (c) 1997-2016 The PHP Group
Zend Engine v3.1.0-dev, Copyright (c) 1998-2016 Zend Technologies
```

> 配置

```bash
cp ~/php-7.0.9/php.ini-development ~/php7/etc/php.ini
cp ~/php7/etc/php-fpm.conf.default ~/php7/etc/php-fpm.conf
cp ~/php7/etc/php-fpm.d/www.conf.default ~/php7/etc/php-fpm.d/www.conf
```

_修改php.ini中的`memory_limit = 512M ,data.timezone=Asia/Chongqing,error_reporting=E_ALL & ~E_NOTICE,upload_max_filesize=20M,default_socket_timeout = 600,opcache.enable=0`等；php-fpm.conf中的`daemonize=yes,error_log=log/php-fpm.log`等;www.conf中的`access.log=log/$pool.access.log,listen = /home/work/php7/var/run/php-fpm.sock`等_

_修改nginx配置，使用unix套接字访问php，如`fastcgi_pass unix:/home/work/php7/var/run/php-fpm.sock;`等_

