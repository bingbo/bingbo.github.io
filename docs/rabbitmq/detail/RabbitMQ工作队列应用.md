工作队列是说由一个生产者发送消息给队列，而可能有一个或多个消费者等待从队列接收并处理消息

__循环调度__

当有多个消费者时，队列循环发送消息给每一个消费者，默认情况下，rabbitmq会按顺序发送一条消息给下一个消费者，平均每个消费者会收到相同数量的消息

__消息确认__

消息确认是防止消息丢失，一个确认信息会从消费者返回，告诉rabbitmq该消费已经收到处理，这时rabbitmq才会释放删除消息。默认情况下消息确认是被关闭的，设置basic_consume()的第四个参数为false(true means no ack),且从消费者发送一个确认即可。

__消息持久化__

要确保消息不丢失我们需要标识队列和消息都是可持久的,首先要确保rabbitmq不会丢失队列，我们需要声明队列为可持久的，这里设置queue_declare的第三个参数为true即可

```php
$channel->queue_declare('task_queue', false, true, false, false);
```

其次我们需要通过delivery_mode = 2参数标记消息是持久化的，如

```php
$msg = new AMQPMessage($data,
   array('delivery_mode' => 2) # make message persistent
);
```

__公平调度__

循环调度容易造成奇偶消息费者闲忙不等的情况，为了公平调度我们可以使用basic_qos方法，同时设置参数prefetch=1,如

```php
$channel->basic_qos(null, 1, null);
```

> 生产者new_task.php

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
//第3个参数为true，使消息队列持久化
$channel->queue_declare('task_queue', false, true, false, false);

$data = implode(' ', array_slice($argv, 1));
if(empty($data)) $data = "Hello World!";
$msg = new AMQPMessage($data,
                            array('delivery_mode' => 2) # 使消息持久化
                                                  );

$channel->basic_publish($msg, '', 'task_queue');

echo " [x] Sent ", $data, "\n";

$channel->close();
$connection->close();
```

> 消费者worker.php

```php

<?php

require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//第3个参数为true，使消息队列持久化
$channel->queue_declare('task_queue', false, true, false, false);

echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

$callback = function($msg) {
      echo " [x] Received ", $msg->body, "\n";
      sleep(substr_count($msg->body, '.'));
      echo " [x] Done", "\n";
      //发送回复确认
      $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
};

//合理分发消息，即等消费者回复确认之后不忙的时候再发送消息
$channel->basic_qos(null,1,null);
//第4个参数为false表示等待消费者回复再删除消息
$channel->basic_consume('task_queue', '', false, false, false, false, $callback);

while(count($channel->callbacks)) {
        $channel->wait();
}
$channel->close();
$connection->close();
```
