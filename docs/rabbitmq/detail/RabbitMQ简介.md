rabbitmq是一个消息代理，它提供了一个平台去发送接收消息，且在你收到消息前你的消息是安全的，这样就可以实现发布订阅、异步处理和工作队列等

### 特性

* 可靠性

    rabbitmq提供了多种技术让你权衡性能与可靠性，包括持久性、投递确认、发布证实和高可用性等。

* 灵活的路由

    消息在到达队列前是通过交换机进行路由的，它提供了多种内置的交换机如：direct,topic,headers,fanout等

* 集群

    多个在同一个局域网中的rabbitmq 服务器可以聚合在一起做为一个单独的逻辑代理使用。

* 联合性

    它提供了联合模型为松散性和非可靠链接。

* 高可用的队列

    队列可以被映射到多个机器中，确保在某些硬件出现故障后消息的安全性。

* 多协议

    支持多种消息协议传递消息

* 多种客户端支持

    支持多种语言的客户端连接，如基于php语言的php-amqplib

* 管理界面

    提供了易用的可视化管理界面，来监控消息代理的每一方面

* 可追踪性

    如果系统出现了问题，提供追踪支持来让你找到问题

* 插件扩展

    rabbitmq附带了各种插件，也可以写自己的扩展插件

### 安装运行

这里基于mac的用brew来安装，其他安装可见官网

```bash
##更新brew版本
brew update
##安装rabbitmq
brew install rabbitmq
```

为当前用户添加rabbitmq环境变量,rabbitmq命令行默认在/usr/local/sbin目录下

```bash
vim .bash_profile
##添加路径
PATH=$PATH:/usr/local/sbin

source .bash_profile

##运行
rabbitmq-server
```

默认用guest用户及guest密码进行本地消息代理服务连接

_参考_

官网：http://www.rabbitmq.com/
