这里会发送消息给多个消费者，而不同的消费者订阅了不同的消息，即发布消息会广播到所有的接收者，但具体接收哪些消息则由接收者去选择。

__Exchanges__

这里涉及到了rabbitmq的整体消息模型，而消息模型的核心概念是生产者不会发送任何消息直接给一个队列，生产者不知道消息是否发送给了任何队列，相应的生产者只是发送消息给了交换机，而交换机必须知道如何处理收到的消息，应该发送消息到哪些队列等，这里有几种不同的交换机类型：direct,topic,headers,fanout,创建交换机如下所示：

```php
$channel->exchange_declare('logs', 'fanout', false, false, false);
```

可以通过下面命令查看所有的已有的交换机`rabbitmqctl list_exchanges`

__临时队列__

不论我们何时连接rabbitmq我们都要先创建一个新的空的队列，为此，我们创建一个随机队列，其次一旦我们断开了与消费者的连接队列就应该自动的删除，如

```php
list($queue_name, ,) = $channel->queue_declare("");
```

此时会产生一个随机名称的队列，如amq.gen-JzTY20BRgKO-HjmUJj0wLg.当连接关闭时队列就会被删除

__绑定__

即告诉交换机要发送消息到相应的队列，交换机与队列之间的关系叫做绑定，如：

```php
$channel->queue_bind($queue_name, 'logs');

你可以通过`rabbitmqctl list_bindings`查看所有的交换机与队列的关系

> 生产者emit_log.php

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//定义扇形类型的logs交换机
$channel->exchange_declare('logs', 'fanout', false, false, false);

$data = implode(' ', array_slice($argv ,1));
if(empty($data)) $data = 'info: hello world!';
$msg = new AMQPMessage($data);

//向logs交换机发送消息
$channel->basic_publish($msg, 'logs');

echo " [x] Sent ", $data, "\n";

$channel->close();
$connection->close();
```

> 消费者receive_log.php

> ```php

<?php
require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//定义扇形类型的logs交换机
$channel->exchange_declare('logs', 'fanout', false, false, false);
//随机生成一个队列
list($queue_name, ,) = $channel->queue_declare('', false, false, true, false);
//把队列绑定到交换机上
$channel->queue_bind($queue_name, 'logs');

echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

$callback = function($msg) {
      echo " [x] Received ", $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, false, false, false, $callback);

while(count($channel->callbacks)) {
        $channel->wait();
}
$channel->close();
$connection->close();
```
