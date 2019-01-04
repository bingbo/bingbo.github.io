## RabbitMQ与java client

### 添加依赖

```xml
<!--rabbitmq-->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>4.0.0</version>
</dependency>
```

### 简单程序应用

```java
public class SimpleRabbitMQ {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception{
        //声明一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("username");
        factory.setPassword("password");
        factory.setVirtualHost("virtualHost");
        factory.setPort(5673);
        //打开自动恢复机制，如果连接失败时
        factory.setAutomaticRecoveryEnabled(true);
        //获取连接
        Connection connection = factory.newConnection();
        //Connection connection1 = factory.newConnection(new Address[]{new Address("192.168.1.4"), new Address("192.168.1.5")});
        Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        String message = "Hello World";
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println(" [x] sent '" + message + "'");

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(" [x] received '" + message + "'");
            }
        };
        channel.basicConsume(QUEUE_NAME, true, consumer);
        channel.close();
        connection.close();
    }
}
```

> 参考文档[rabbit-api-guide](http://www.rabbitmq.com/api-guide.html) [tutorials](http://www.rabbitmq.com/getstarted.html)

## RabbitMQ与Spring集成

该部分包括两部分，`spring-amqp`是基础抽象，而`spring-rabbit`是RabbitMQ的实现,有如下特性：

* 用于异步处理入站消息的侦听器容器
* RabbitTemplate用于发送和接收消息
* RabbitAdmin用于自动声明队列，交换和绑定

### 添加依赖

> 目前依赖为1.6.6的发布版本

```xml
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>1.6.6.RELEASE</version>
</dependency>
```

### 简单程序实现

```java

public class AmqJava {

    public static void main(String[] args) throws Exception {
        //定义一个连接工厂
        ConnectionFactory factory = new CachingConnectionFactory("localhost");
        //基于该连接的管理操作类,用于声明队列、交换和绑定
        RabbitAdmin admin = new RabbitAdmin(factory);
        //定义一个消息队列
        Queue queue = new Queue("myqueue");
        //声明绑定一个队列
        admin.declareQueue(queue);
        //定义一个交换主题
        TopicExchange topicExchange = new TopicExchange("myExchange");
        //声明绑定一个主题
        admin.declareExchange(topicExchange);
        //具体的绑定操作
        admin.declareBinding(BindingBuilder.bind(queue).to(topicExchange).with("foo.*"));
        //监听器容器
        SimpleMessageListenerContainer listenerContainer = new SimpleMessageListenerContainer(factory);
        Object listener=new Object(){
            public void handleMessage(String foo){
                System.out.println(foo);
            }
        };
        //消息监听适配器，通过反射机制调用目标监听器的方法来处理消息，并进行灵活的消息类型转换
        MessageListenerAdapter adapter = new MessageListenerAdapter(listener);
        listenerContainer.setMessageListener(adapter);
        listenerContainer.setQueueNames("myqueue");
        listenerContainer.start();

                                                                                                                        //辅助类，简化同步RabbitMQ访问（发送和接收消息）
        RabbitTemplate template = new RabbitTemplate(factory);
        template.convertAndSend("myExchange", "foo.bar", "Hello, world!");
        template.convertAndSend("myExchange", "foo.say", "Hi, world!");
        Thread.sleep(1000);
        listenerContainer.stop();
                                                                                                                    }
}
```


### 通过Spring配置实现

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xmlns:rabbit="http://www.springframework.org/schema/rabbit"
                     xsi:schemaLocation="http://www.springframework.org/schema/beans
                     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd






http://www.springframework.org/schema/rabbit  http://www.springframework.org/schema/rabbit/spring-rabbit-1.6.xsd"


>


<rabbit:connection-factory id="connectionFactory" host="localhost"/>

<rabbit:template id="amqpTemplate" connection-factory="connectionFactory"
                 exchange="myExchange" routing-key="foo.bar"/>

<rabbit:admin connection-factory="connectionFactory"/>

<rabbit:queue name="myQueue"/>

<rabbit:topic-exchange name="myExchange">
    <rabbit:bindings>
         <rabbit:binding queue="myQueue" pattern="foo.*"/>
    </rabbit:bindings>
</rabbit:topic-exchange>


<rabbit:listener-container connection-factory="connectionFactory">
     <rabbit:listener ref="foo" method="listen" queue-names="myQueue"/>
</rabbit:listener-container>

<bean id="foo" class="com.ibingbo.rabbitmq.Foo"/>


</beans>
```

#### java程序调用

```java
public class AmqSpringWay {
    public static void main(String[] args) throws Exception {
        AbstractApplicationContext context = new ClassPathXmlApplicationContext("spring-rabbitmq-conf.xml");
        RabbitTemplate template = context.getBean(RabbitTemplate.class);
        template.convertAndSend("Hello, spring amq " );
        Thread.sleep(1000);
        context.destroy();
    }
}
```

```java
//简单消息处理器
public class Foo {

    public void listen(String foo) {
        System.out.println("Foo listenner receive : "+foo);
    }
}
```

> 参考文档[spring-amqp](http://projects.spring.io/spring-amqp/)
