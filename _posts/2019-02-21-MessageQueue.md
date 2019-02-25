---
layout: post
title:  "Backend | Message Queue"
date:   2019-02-021 17:11
author: Botao Xiao
comment: true
categories: JavaBackend
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
#### Hello world(Simple) Pattern
1. Hello world pattern binds a single producer, consumer to one message queue.
![simple pattern](https://i.imgur.com/OSN2nqc.png)

#### Work queue Pattern
1. Single producer and multiple consumers are binging to a single message queue.
2. Work queue can be applied to I/O bound task.
![work queue](https://i.imgur.com/I1i2Fo6.png)

#### Publish/Subscribe Pttern
1. In order to produce publish/subscribe, we need to have multiple message queue connected to different consumers. We also have a single producer and it will send message to an exchange, and exchange will put the message to corresponding message queue and its consumer will handle this message.
![Publish / subscribe pattern](https://i.imgur.com/b73ppOs.png)

#### Routing pattern
1. Producer create a message and bind a exchange type to this message. Very similar as Publish / Subscribe pattern, we also have multiple message queues and producers binding to the exchange.
2. Exchange will decode the message from producer and read the binding key, it will use the binding key to select which message queue message is going to.
3. The message queue is specified and no regex is included.
![Routing pattern](https://i.imgur.com/04CE5PM.png)

#### Topic pattern
1. Different from routing pattern, we can use regex to decode the binding key.
    * #: match multiple word.
    * \*: used to match single word.
![Topic pattern](https://i.imgur.com/X7E4n1E.png)

### Reference
1. [消息队列RabbitMQ入门与5种模式详解](https://www.jianshu.com/p/80eefec808e5)