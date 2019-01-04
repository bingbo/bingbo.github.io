通过路由可以针对消费者所接收的消息进行进一步的过滤，相当于是对消息进行分类，生产者可以发送任何种类的消息，而不同的消费者可以接收不同类型的消息

__绑定__

这里的绑定多加一个路由key参数来对消息进行分类过滤等，如：

```php
$binding_key = 'black';
$channel->queue_bind($queue_name, $exchange_name, $binding_key);
```

__直连交换机(direct exchange)__

直连交换机的路由算法很简单，即一个消息将发往binding key与消息的路由key相匹配的队列里

__多个绑定__

即可以由多个队列根据不同的binding key绑定到一个交换机上，也就是说一个消费者可能只接收一种消息，另一个消费者可能接收所有的消息

> 生产者emit_log_direct.php

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//定义直连类型的direct_logs交换机
$channel->exchange_declare('direct_logs', 'direct', false, false, false);
$serverity = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'info';

$data = implode(' ', array_slice($argv ,2));
if(empty($data)) $data = 'info: hello world!';
$msg = new AMQPMessage($data);

//向direct_logs交换机发送$serverity消息
$channel->basic_publish($msg, 'direct_logs', $serverity);

echo " [x] Sent ", $serverity,':',$data, "\n";

$channel->close();
$connection->close();
```

> 消费者receive_logs_direct.php

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//定义直连类型的direct_logs交换机
$channel->exchange_declare('direct_logs', 'direct', false, false, false);
//随机生成一个队列
list($queue_name, ,) = $channel->queue_declare('', false, false, true, false);
$serverities = array_slice($argv,1);
if(empty($serverities)){
    exit(1);
}
foreach($serverities as $serverity){
    //把队列绑定到交换机上
    $channel->queue_bind($queue_name, 'direct_logs',$serverity);
}


echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

$callback = function($msg) {
      echo ' [x] ',$msg->delivery_info['routing_key'], ':', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, false, false, false, $callback);

while(count($channel->callbacks)) {
        $channel->wait();
}
$channel->close();
$connection->close();
```

可以接收warning和error消息到一个文件里

```bash
php receive_logs_direct.php warning error > logs_from_rabbit.log
```

可以接收所有的消息到屏幕上显示 

```bash
php receive_logs_direct.php info warning error
 [*] Waiting for logs. To exit press CTRL+C
```

发送error消息

```bash
php emit_log_direct.php error "Run. Run. Or it will explode."
 [x] Sent 'error':'Run. Run. Or it will explode.'
```
