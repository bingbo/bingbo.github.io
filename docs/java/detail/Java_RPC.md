## RPC一般流程

1. 服务消费方（client）调用以本地调用方式调用服务；
1. client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；
1. client stub找到服务地址，并将消息发送到服务端；
1. server stub收到消息后进行解码；
1. server stub根据解码结果调用本地的服务；
1. 本地服务执行并将结果返回给server stub；
1. server stub将返回结果打包成消息并发送至消费方；
1. client stub接收到消息，并进行解码；
1. 服务消费方得到最终结果。


### 流程图

![](https://github.com/bingbo/blog/blob/master/images/RPC%E6%B5%81%E7%A8%8B.jpg)

## RPC部署结构

![](https://github.com/bingbo/blog/blob/master/images/RPC%E6%9C%8D%E5%8A%A1%E9%83%A8%E7%BD%B2.jpg)
