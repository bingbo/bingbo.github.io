在远程机器上运行一个函数并等待响应结果，这个情况一般认为是远程过程调用即RPC

__回调队列__

一般是客户端发送请求消息而服务端回复响应消息，为了能收到响应我们需要发送回调队列在请求里，如下所示：

```php
list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$msg = new AMQPMessage(
            $payload,
                array('reply_to' => $queue_name));

$channel->basic_publish($msg, '', 'rpc_queue');
```

__Correlation Id__

相关性ID，用来作为每一个请求的唯一标识，这样当我们从回调队列里收到消息时，根据此属性我们就能知道是哪个请求的响应了，如果没有相关性ID，则丢弃消息

_RPC流程为_

1. 当客户端启动时，会创建一个匿名的回调队列
2. 针对一个RPC请求，客户端会发送两个属性：reply_to，用来标识回调队列；correlation_id，用来标识唯一的请求
3. 请求被发送到rpc_queue队列
4. RCP工作者也即服务器等待该队列的请求，当一个请求出现时它就会处理请求并通过reply_to指定的队列发送消息结果给客户端
5. 客户端等待回调队列的数据，当一个消息出现时，它会检查correlation_id，如果匹配则返回响应

> 服务端rcp_server.php

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->queue_declare('rpc_queue', false, false, false, false);

function fib($n) {
    if ($n == 0)
        return 0;
    if ($n == 1)
        return 1;
    return fib($n-1) + fib($n-2);
}

echo " [x] Awaiting RPC requests\n";
$callback = function($req) {
    $n = intval($req->body);
    echo " [.] fib(", $n, ")\n";

    $msg = new AMQPMessage(
        (string) fib($n),
        array('correlation_id' => $req->get('correlation_id'))
        );

    $req->delivery_info['channel']->basic_publish(
        $msg, '', $req->get('reply_to'));
    $req->delivery_info['channel']->basic_ack(
        $req->delivery_info['delivery_tag']);
};

$channel->basic_qos(null, 1, null);
$channel->basic_consume('rpc_queue', '', false, false, false, false, $callback);

while(count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

> 客户端rpc_client.php

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class FibonacciRpcClient {
    private $connection;
    private $channel;
    private $callback_queue;
    private $response;
    private $corr_id;

    public function __construct() {
        $this->connection = new AMQPStreamConnection(
            'localhost', 5672, 'guest', 'guest');
        $this->channel = $this->connection->channel();
        list($this->callback_queue, ,) = $this->channel->queue_declare(
            "", false, false, true, false);
        $this->channel->basic_consume(
            $this->callback_queue, '', false, false, false, false,
            array($this, 'on_response'));
    }
    public function on_response($rep) {
        if($rep->get('correlation_id') == $this->corr_id) {
            $this->response = $rep->body;
        }
    }

    public function call($n) {
        $this->response = null;
        $this->corr_id = uniqid();

        $msg = new AMQPMessage(
            (string) $n,
            array('correlation_id' => $this->corr_id,
                  'reply_to' => $this->callback_queue)
            );
        $this->channel->basic_publish($msg, '', 'rpc_queue');
        while(!$this->response) {
            $this->channel->wait();
        }
        return intval($this->response);
    }
};

$fibonacci_rpc = new FibonacciRpcClient();
$response = $fibonacci_rpc->call(30);
echo " [.] Got ", $response, "\n";

```

启动服务器等待请求

```bash
 php rpc_server.php
  [x] Awaiting RPC requests
```

发送请求并等待响应

```bash
php rpc_client.php
 [x] Requesting fib(30)
```
