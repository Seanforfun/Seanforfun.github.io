---
layout: post
title:  "Spring Cloud Bus 消息总线"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: SpringCloud
description: 消息队列可以简单地使用FIFO队列实现，但是消息队列是存在在栈中的，只有一个进程内的线程才能取得消息。在微服务中，业务被分布在多个服务之间，所以我们使用消息总线机制来实现多个微服务之间的通讯。
---
### 消息总线和消息队列
消息队列可以简单地使用FIFO队列实现，但是消息队列是存在在栈中的，只有一个进程内的线程才能取得消息。

在微服务中，业务被分布在多个服务之间，所以我们使用消息总线机制来实现多个微服务之间的通讯。

### 消息总线构架图
![消息总线构架图](https://i.imgur.com/QQnheXi.png)
1. Config-client, Config-server被注册成为服务并被server程序发现。
2. Config-config被注册成为配置服务器，通过@EnableConfigServer,并通过配置文件连接到远程端的配置文件，并在config-client中配置config-server的信息，client可以通过和config-server交互，并从远端获得配置信息。可以参考[Spring Cloud Config 配置文件](https://github.com/Seanforfun/JavaCore/blob/master/Conclusions/SpringCloudConfig.md)。
3. 在远程端更新消息，此时任意一个用户端通过http://localhost:8881/actuator/bus-refresh 请求（该请求必须是POST请求），更新的消息通过RabbitMQ将更新的消息传递到整个集群。实现了无需重启即更新集群。

### Config-Client的实现
1. 在config-client服务中配置RabbitMQ信息。
```Properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.cloud.bus.enabled=true
spring.cloud.bus.trace.enabled=true
management.endpoints.web.exposure.include=bus-refresh
```

2. 通过添加@RefreshScope注解，确定每次更新的域，每次更新时Spring容器将会检测这些域并更新其中的@Value定义的值。
```Java
@RestController
@RefreshScope
public class HiController {
    @Value("${foo}")
    private String foo;
    @RequestMapping("/hi")
    public String SayHello(){
        return this.foo;
    }
}
```

3. 通过POST请求，我们向其中任意一个Config-client发送更新请求。
![POST发送更新请求](https://i.imgur.com/2hcFuiq.png)

### Result
再次请求后会发现已经被更新：
foo version 4

### Reference
1. [SpringCloud教程第8篇：Spring Cloud Bus（F版本）](https://www.fangzhipeng.com/springcloud/2018/08/30/sc-f8-bus/)