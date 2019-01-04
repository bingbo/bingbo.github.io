rabbitmq作为一个消息队列代理，可以实现接收、存储和发送消息，由生产者发送消息给指定队列，队列接收并存储消息，消费者等待并从队列中取出消息

__生产者__

仅仅用来发送消息，即一个发送消息的程序，我们叫做"P"

__队列__

队列类似于邮箱，它常住rabbitmq内部，队列不受任何限制，可以任何多的数据，实质是一个缓存，多个生产者可以发送消息到队列，也可以多个消费者从一个队列接收消息

__消费者__

一个等待接收消息的程序

> 生产者send.php

```php
<?php

require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

//连接消息服务器
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//创建一个队列
$channel->queue_declare('hello', false, false, false, false);

//创建并发送消息
$msg = new AMQPMessage('Hello World!');
$channel->basic_publish($msg, '', 'hello');

echo " [x] Sent 'Hello World!'\n";

$channel->close();
$connection->close();
```

> 消费者receive.php

```php
<?php

require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

//连接消息服务器
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//创建一个队列
$channel->queue_declare('hello', false, false, false, false);

echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

$callback = function($msg) {
      echo " [x] Received ", $msg->body, "\n";
};

//接收队列消息
$channel->basic_consume('hello', '', false, true, false, false, $callback);

//循环等待消息
while(count($channel->callbacks)) {
        $channel->wait();
}
```
