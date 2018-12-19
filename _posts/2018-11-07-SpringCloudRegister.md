---
layout: post
title:  "Spring Cloud注册与发现：Eureka"
date:   2018-11-07 08:43:59
author: Botao Xiao
comment: true
categories: SpringCloud
description: 注册与发现又可以被理解为服务的治理，我们通过这个构架可以实现各个微服务的自动化的注册与发现。主要分成两个步骤：服务注册和服务发现。
---
注册与发现又可以被理解为服务的治理，我们通过这个构架可以实现各个微服务的自动化的注册与发现。主要分成两个步骤：服务注册和服务发现。Spring cloud 二次封装了Netflix的Erureka项目，实现了注册与发现。

## 基础概念
### 服务提供者
#### 服务注册
1. 提供者在启动时会发送REST请求将自己注册到Eureka server上。Eureka Server接收到REST请求后，会解析元数据，格式为<服务名， <私有服务名， 服务>>。
2. 可以通过```eureka.client.register-with-eureka=false```来选择当前服务不要注册到注册中心上。

#### 服务同步
1. 服务被注册到多个注册中心上，而两个注册中心之间也是相互注册的，当一个注册中心发现了注册的服务时，会将服务同步到所有的应该被注册的注册中心上。换句话说，就是服务会被自动同步。

#### 服务续约
1. 当一个服务被注册后，服务的提供者会维护一个心跳来保持和注册中心的联系。这就是服务的续约。
```Properties
#30秒回发送一次心跳保活。
eureka.instance.lease-renewal-interval-in-seconds=30
#90秒没有收到心跳意味着当前服务已经失效。
eureka.instance.lease-expiration-duration-in-seconds=90
```

### 服务消费者
#### 获取服务
1. 此时注册中心已经被注册了一些服务，消费者会向服务注册中心发送一个请求，获取服务的列表。
2. 获取服务是服务消费者的基础，通过```eureka.client.fetch-registry=true```来确保服务获取功能被开启。

#### 服务调用
1. 服务消费者在获取服务清单后，可以通过服务名获得具体服务的实例名和该实例的元数据信息，决定调用哪一个服务实例。

#### 服务下线
当服务下线后，服务实例会触发一个服务下线的REST请求给Eureka server。服务端接收到这个事件后，会将服务的状态值为DOWN，并将下线时间传播出去。

### 服务注册中心
#### 失效剔除
当服务因为非正常原因下线，注册中心并未收到服务下线，注册中心会将这些无法提供服务的实例剔除。

#### 自我保护
Eureka Server的运行期间，会统计心跳失败的比例是否低于85%，Eureka Server会将这些服务保护起来，服务不过期，这种情况大多数情况下发生在本地开发的时候。可以使用```eureka.server.enable-self-preservation=false```来关闭。

## Demo
### 创建Server和Client
创建两个SpringBoot项目，一个作为Server，一个作为Client。选用Eureka作为服务器。

#### 创建SpringCloudServer
1. 在SpringBoot项目的启动类中，添加注解@EnableEurekaServer
```Java
@SpringBootApplication
@EnableEurekaServer
public class SpringCloudServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudServerApplication.class, args);
	}
}
```

2. 定义配置文件application.properties
```Properties
server.port=1111

eureka.instance.hostname=localhost
# Current service works as a server so we don't register itself to the register center.
eureka.client.register-with-eureka=false
# Current service (as a client) doesn't need to search for other registered services.
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka
```

#### 创建SpringCloudClient
1. 在SpringBootClient的启动类中，添加@EnableEurekaClient
```Java
@SpringBootApplication
@EnableDiscoveryClient
public class ClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ClientApplication.class, args);
    }
}
```

2. 定义properties文件
```Properties
server.port=1112 # 当前客户服务占用的端口
spring.application.name=hello-service   # 当前服务的名字
eureka.client.service-url.defaultZone=http://localhost:1111/eureka  # 指定服务注册中心的地址
```

### 结果（old version）
![Spring Cloud](https://i.imgur.com/UVakJZt.png)
在Server的页面中可以找到Service-hi，说明已经成功注册服务了。

## 高可用注册中心
1. 高可用注册中心的原理：包括注册中心，每个结点均是服务的提供方，也是服务的消费方。
2. 注册中心将自己作为一个服务的提供方，将自己的配置信息注册到别的注册中心上，实现了服务清单的同步。
3. 在微服务的分布式构架中，我们需要充分考虑发生故障的情况。使用高可用注册中心有助于我们在某个服务中心宕机以后仍能支撑整个微服务构架。

### 创建两个服务注册中心，并向对方注册自己
1. peer1注册中心， 其中peer1, peer2...都是在C:\Windows\System32\drivers\etc\hosts文件中定义的。
```Properties
# 当前注册中心的端口地址和项目名
server.port=1111
spring.application.name=eureka-server
# 当前服务中心要注册的地址
# 将当前服务注册到peer2上
eureka.instance.hostname=peer2
eureka.client.serviceUrl.defaultZone=http://peer2:1112/eureka/
```

2. peer2注册中心
```Java
server.port=1112
spring.application.name=eureka-server
# 将当前的服务注册到peer1上
eureka.instance.hostname=peer1
eureka.client.serviceUrl.defaultZone=http://peer1:1111/eureka
```

3. 结果：此处我使用了三个注册中心的高可用
![Imgur](https://i.imgur.com/hAaKL1S.png)

4. 将服务的提供者注册到多个服务器上（Optional）
```properties
server.port=1114
spring.application.name=hello-service
# 此时服务的提供者已经被注册到了三个注册中心上。
eureka.client.service-url.defaultZone=http://peer1:1111/eureka, http://peer1:1112/eureka, http://peer3:1113/eureka
```

5. 通过ip地址表示注册中心地址。（Optional）
eureka.instance.prefer-ip-address=true

### Reference
1. [SpringCloud教程第1篇：Eureka（F版本）](https://www.fangzhipeng.com/springcloud/2018/08/30/sc-f1-eureka/)
