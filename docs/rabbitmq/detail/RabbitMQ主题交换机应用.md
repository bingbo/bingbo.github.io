这里发送给主题交换机的消息没一个确定的routing_key,它必须是一系统的单词且由.分隔的，而单词可以是随意的但一般都是有示意的，如"stock.usd.nyse","nyse.vmw"等，这里的routing_key可以有多个单组成，但不能超过255字节。

binding key也必须是一样的形式，其原理跟direct交换机是一样的，消息会被发送到其binding key与routing_key相匹配的队列，这里有两种特殊的binding key:

* *(star) 只代表一个单词
* #(hash) 代表0个或多个单词

当一个队列绑定的是"#"binding key，则它将收到所有的消息，而忽略路由key，就像fanout交换机一样；当"*"和"#"都没有用于bindings key时，则topic交换机与direct交换机一样

> 生产者emit_log_topic.php

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//定义主题类型的topic_logs交换机
$channel->exchange_declare('topic_logs', 'topic', false, false, false);
$routing_key = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'anonymous.info';

$data = implode(' ', array_slice($argv ,2));
if(empty($data)) $data = 'info: hello world!';
$msg = new AMQPMessage($data);

//向topic_logs交换机发送$serverity消息
$channel->basic_publish($msg, 'topic_logs', $routing_key);

echo " [x] Sent ", $routing_key,':',$data, "\n";

$channel->close();
$connection->close();
```

> 消费者receive_log_topic.php

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//定义直连类型的topic_logs交换机
$channel->exchange_declare('topic_logs', 'topic', false, false, false);
//随机生成一个队列
list($queue_name, ,) = $channel->queue_declare('', false, false, true, false);
$binding_keys = array_slice($argv,1);
if(empty($binding_keys)){
    exit(1);
}
foreach($binding_keys as $binding_key){
    //把队列绑定到交换机上
    $channel->queue_bind($queue_name, 'topic_logs',$binding_key);
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

> 接收所有的日志

```bash
php receive_logs_topic.php "#"
```

> 接收"kern"相关的日志

```bash
php receive_logs_topic.php "kern.*"
```

> 接收多种日志数据

```bash
php receive_logs_topic.php "kern.*" "*.critical"
```

> 发送"kern.critical"类型的消息

```bash
php emit_log_topic.php "kern.critical" "A critical kernel error"
```
