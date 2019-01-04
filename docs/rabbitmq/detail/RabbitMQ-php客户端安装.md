rabbitmq针对php的客户端目前常用的是php-amqplib

有以下两种安装使用方法

__直接从github上把相应的库文件拉下来使用__

github地址：https://github.com/php-amqplib/php-amqplib

```bash
##直接git clone即可
git clone https://github.com/php-amqplib/php-amqplib.git
```

__用composer进行下载安装__

在一个目录中添加composer.json文件，如

```javascript
{
    "require": {
            "php-amqplib/php-amqplib": "2.5.*"
    }
}
```

然后运行以下命令

```bash
composer install
```

