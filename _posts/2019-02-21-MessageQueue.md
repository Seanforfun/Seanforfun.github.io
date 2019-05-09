---
layout: post
title:  "Backend | Message Queue"
date:   2019-02-21 17:11
author: Botao Xiao
comment: true
categories: Backend
description: Message queue is an asynchronous message communication between different micro services. Message will be saved on message queue before handled by a service. The message queue can be used to deffer the high periodical volumn.
---
# Backend | Message Queue

Message queue is an asynchronous message communication between different micro services. Message will be saved on message queue before handled by a service. The message queue can be used to deffer the high periodical volumn.

### Concept explaination
1. Server(Broker): Server receives messages come from clients, it will implements the function of AMQP protocol and router.
2. Virtual Host: A group of exchange and queue, there is a administrator for each virtual host.
3. Exchange: Receive messages from producer, and it will decode the routing key and send the message to a correct queue.
4. ExchangeType: The strategy of exchange to send the message to different queues.
5. Message queue: A container to save all unhandled messages.
6. Message: A message contains two parts, header and body.
7. Binding key: Bingding key is used to connect a exchange and a queue.

### Patterns of Message Queue
#### Direct Pattern
1. Hello world pattern binds a single producer, consumer to one message queue.
![simple pattern](https://i.imgur.com/OSN2nqc.png)

#### Work queue Pattern
1. Single producer and multiple consumers are binging to a single message queue.
2. Work queue can be applied to I/O bound task.
![work queue](https://i.imgur.com/I1i2Fo6.png)

#### Fanout(Publish/Subscribe) Pttern
1. In order to produce publish/subscribe, we need to have multiple message queue connected to different consumers. We also have a single producer and it will send message to an exchange, and exchange will put the message to corresponding message queue and its consumer will handle this message.
![Publish / subscribe pattern](https://i.imgur.com/b73ppOs.png)

#### Routing pattern
1. Producer create a message and bind a exchange type to this message. Very similar as Publish / Subscribe pattern, we also have multiple message queues and producers binding to the exchange.
2. Exchange will decode the message from producer and read the binding key, it will use the binding key to select which message queue message is going to.
3. The message queue is specified and no regex is included.
![Routing pattern](https://i.imgur.com/04CE5PM.png)

#### Topic pattern
1. Different from routing pattern, we can use regex to decode the binding key.
    * match multiple word.
    * used to match single word.
![Topic pattern](https://i.imgur.com/X7E4n1E.png)

#### Header pattern
1. Header pattern will ignore the binding key and use a map as authentication token.
2. Producer will insert a map as token and consumer need to create the same map and compare them, if same, receiver will load message from queue.

### Rabbit MQ Implemetation
Before implementing Rabbit MQ with spring, we need to prepare the pox.xml and properties file in applcation.properties.
1. Inject the dependency
```Xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
</dependencies>
```

2. Configure the properties file
```Properties
spring.rabbitmq.host=${server.address}  # Address of rabbit mq running on
spring.rabbitmq.port=5672               # Port number of rabbit mq running on
spring.rabbitmq.username=guest          # Username
spring.rabbitmq.password=guest
spring.rabbitmq.publisher-confirms=true # Whether to enable publisher confirms.
spring.rabbitmq.virtual-host=/          # As mentioned in previous part, we can configure virtual-host by using this setting.
spring.rabbitmq.listener.simple.concurrency= 10 # Minimum number of listener invoker threads.
spring.rabbitmq.listener.simple.max-concurrency= 100 # Maximum number of listener
spring.rabbitmq.listener.simple.prefetch= 1 # Maximum number of unacknowledged messages that can be outstanding at each consumer.
spring.rabbitmq.listener.simple.auto-startup=true # Whether to start the container automatically on startup.
spring.rabbitmq.listener.simple.default-requeue-rejected= true
spring.rabbitmq.template.retry.enabled=true # Whether publishing retries are enabled.
spring.rabbitmq.template.retry.initial-interval=1000ms # Duration between the first and second attempt to deliver a message.
spring.rabbitmq.template.retry.max-attempts=3 # Maximum number of attempts to deliver a message.
spring.rabbitmq.template.retry.max-interval=10000ms # Maximum duration between attempts.
spring.rabbitmq.template.retry.multiplier=1.0 # Multiplier to apply to the previous retry interval.
spring.rabbitmq.publisher-confirms=true # Whether to enable publisher confirms.
spring.rabbitmq.listener.direct.acknowledge-mode=manual # Acknowledge mode of container.
spring.rabbitmq.listener.simple.acknowledge-mode=manual # # Acknowledge mode of container.
```

#### Acknowledge-mode
acknowledge-mode's three options: None, Manual and Auto
    * None: Sender will not require a ack from receiver and receiver won't send ack as well.
    * Manual: Sender requires a ack from receiver and programmer needs to manually send ack.
    * Auto: Sender requires a ack from receiver and receiver will automatically send ack, if exception happens, it will throw it and won't give a ack back to sender.
    
#### Direct Pattern(Manual ack)
1. Configure a queue as message queue.
```Java
public static final String DIRECT_QUEUE = "DIRECT QUEUE";
@Bean
public Queue directQueue(){
    return new Queue(DIRECT_QUEUE, true);
}
```

2. Message sender
```Java
public class MqSender {
    @Autowired
    private AmqpTemplate amqpTemplate;  // ampyTemplate will be injected.
    public void send(Object message){
        String msgString = RedisService.beanToString(message);
        log.info("[Sender]: send message", msgString);
        amqpTemplate.convertAndSend(MqConfigure.DIRECT_QUEUE, msgString);   // Send message to message queue.
    }
}
```

3. Message receiver
```Java
public class MqReceiver {
    @RabbitListener(queues = {MqConfigure.DIRECT_QUEUE})    // Listen to the messages from this message queue.
    @RabbitHandler  //Register a handler for receiving message
    public void receive(String receive, Channel channel, Message message) throws IOException {
        log.info("[Receiver]: receive {}", receive);
        try {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);   // If success, send an ack back to sender
        } catch (IOException e) {
            e.printStackTrace();
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);  // if catch an exception, send a nack back, and the last parameter false means this message is dropped.
        }
    }
}
```

4. Results
```Java
2019-02-22 12:56:35.698  INFO 14960 --- [0.1-8080-exec-1] io.seanforfun.seckill.rabbitmq.MqSender  : [Sender]: send message
2019-02-22 12:56:35.699  INFO 14960 --- [0.1-8080-exec-1] io.seanforfun.seckill.rabbitmq.MqSender  : [Sender]: send message
2019-02-22 12:56:35.701  INFO 14960 --- [cTaskExecutor-4] i.s.seckill.rabbitmq.MqReceiver          : [Receiver]: receive Hello world!
2019-02-22 12:56:35.701  INFO 14960 --- [cTaskExecutor-1] i.s.seckill.rabbitmq.MqReceiver          : [Receiver]: receive Hello world!
```

### Work queue pattern(Single queue, multiple consumer)
1. Configure a queue as a message queue(Same as direct pattern)
```Java
@Bean
public Queue directQueue(){
    return new Queue(DIRECT_QUEUE, true);
}
```

2. Sender
```Java
public class MqSender {
    @Autowired
    private AmqpTemplate amqpTemplate;
    public void send(Object message){
        String msgString = RedisService.beanToString(message);
        log.info("[Sender]: send message", msgString);
        amqpTemplate.convertAndSend(MqConfigure.DIRECT_QUEUE, msgString);
    }
}
```

3. Multiple receiver
```Java
public class MqReceiver {
    public AtomicInteger count = new AtomicInteger(0);
    @RabbitListener(queues = {MqConfigure.DIRECT_QUEUE})
    @RabbitHandler
    public  void receive(String receive, Channel channel, Message message) throws IOException {
        log.info("[Receiver0]: receive {}", receive);
        try {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println(count.getAndIncrement());
        } catch (IOException e) {
            e.printStackTrace();
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
        }
    }
    @RabbitListener(queues = {MqConfigure.DIRECT_QUEUE})
    @RabbitHandler
    public void receive1(String receive, Channel channel, Message message) throws IOException {
        log.info("[Receiver1]: receive {}", receive);
        try {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println(count.getAndIncrement());
        } catch (IOException e) {
            e.printStackTrace();
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
        }
    }
}
```

4. Result: Too long to list, we have both receiver0 and receiver1 will handle the request.

#### Fanout pattern
1. Configure several queues and a exchange, we need to  bind the exchange with queue.
```Java
/**
 * Fanout pattern
 * @return
 */
@Bean
public Queue fanoutQueue1(){
    return new Queue(FANOUT_QUEUE_1, true);
}
@Bean
public Queue fanoutQueue2(){
    return new Queue(FANOUT_QUEUE_2, true);
}
@Bean
public FanoutExchange fanoutExchange(){
    return new FanoutExchange(FANOUT_EXCHANGE);
}
@Bean
public Binding fanoutQueue1Bind(){
    return BindingBuilder.bind(fanoutQueue1()).to(fanoutExchange());
}
@Bean
public Binding fanoutQueue2Bind(){
    return BindingBuilder.bind(fanoutQueue2()).to(fanoutExchange());
}
```

2. Sender: We don't send message to exchange instead of queue, the queue will broadcast message to all consumers.
```Java
public void fanoutSender(Object message){
    String msgString = RedisService.beanToString(message);
    log.info("[Sender]: send message", msgString);
    amqpTemplate.convertAndSend(MqConfigure.FANOUT_EXCHANGE, "", msgString);
}
```

3. Receiver
```Java
public class MqReceiver {
    public AtomicInteger count = new AtomicInteger(0);

    @RabbitListener(queues = {MqConfigure.FANOUT_QUEUE_1})
    @RabbitHandler
    public  void receive(String receive, Channel channel, Message message) throws IOException {
        log.info("[Receiver0]: receive {}", receive);
        try {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println(count.getAndIncrement());
        } catch (IOException e) {
            e.printStackTrace();
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
        }
    }
    @RabbitListener(queues = {MqConfigure.FANOUT_QUEUE_2})
    @RabbitHandler
    public void receive1(String receive, Channel channel, Message message) throws IOException {
        log.info("[Receiver1]: receive {}", receive);
        try {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println(count.getAndIncrement());
        } catch (IOException e) {
            e.printStackTrace();
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
        }
    }
}
```

#### Result
```
2019-02-22 14:12:23.843  INFO 15532 --- [0.1-8080-exec-1] io.seanforfun.seckill.rabbitmq.MqSender  : [Sender]: send message
2019-02-22 14:12:23.858  INFO 15532 --- [cTaskExecutor-9] i.s.seckill.rabbitmq.MqReceiver          : [Receiver0]: receive Hello world!
2019-02-22 14:12:23.858  INFO 15532 --- [cTaskExecutor-1] i.s.seckill.rabbitmq.MqReceiver          : [Receiver1]: receive Hello world!
```

### Routing/Topic pattern
1. Configure message queue, exchange, we need to define the routeKey when binding queue to exchange.
```Java
/**
 * Topic pattern
 * @return
 */
@Bean
public Queue topicQueue1(){
    return new Queue(TOPIC_QUEUE_1, true);
}
@Bean
public Queue topicQueue2(){
    return new Queue(TOPIC_QUEUE_2, true);
}
@Bean
public TopicExchange topicExchange(){
    return new TopicExchange(TOPIC_EXCHANGE);
}
@Bean
public Binding topicQueue1Bind(){
    return BindingBuilder.bind(topicQueue1()).to(topicExchange()).with("test.*");
}
@Bean
public Binding topicQueue2Bind(){
    return BindingBuilder.bind(topicQueue2()).to(topicExchange()).with("test.test1");
}
```

2. Sender
```Java
 public void topicSender(Object message){
    String msgString = RedisService.beanToString(message);
    log.info("[Sender]: send message", msgString);
    amqpTemplate.convertAndSend(MqConfigure.TOPIC_EXCHANGE, "test.test1", msgString);
}
```

3. Receiver
```Java
@RabbitListener(queues = {MqConfigure.TOPIC_QUEUE_1})
@RabbitHandler
public void receive2(String receive, Channel channel, Message message) throws IOException {
    log.info("[Receiver0]: receive {}", receive);
}
@RabbitListener(queues = {MqConfigure.TOPIC_QUEUE_2})
@RabbitHandler
public void receive3(String receive, Channel channel, Message message) throws IOException {
    log.info("[Receiver1]: receive {}", receive);
}
```

4. Result
```
2019-02-22 14:32:42.938  INFO 12644 --- [0.1-8080-exec-1] io.seanforfun.seckill.rabbitmq.MqSender  : [Sender]: send message
2019-02-22 14:32:42.947  INFO 12644 --- [cTaskExecutor-1] i.s.seckill.rabbitmq.MqReceiver          : [Receiver1]: receive Hello world!
2019-02-22 14:32:42.947  INFO 12644 --- [cTaskExecutor-1] i.s.seckill.rabbitmq.MqReceiver          : [Receiver0]: receive Hello world!
```

### Reference
1. [消息队列RabbitMQ入门与5种模式详解](https://www.jianshu.com/p/80eefec808e5)
2. [AMQP 0-9-1 Model Explained](https://www.rabbitmq.com/tutorials/amqp-concepts.html)















