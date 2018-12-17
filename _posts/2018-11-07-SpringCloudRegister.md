---
layout: post
title:  "Spring Cloud注册与发现"
date:   2018-11-07 08:43:59
author: Botao Xiao
comment: true
categories: SpringCloud
description: 
---
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
server.port=8761	# 定义Server的启动端口
eureka.instance.hostname=Seanforfuna---
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
1. Eureka服务端（服务注册中心）：作为注册的中心，存储注册的服务，支持高可用配置。
2. Eureka客户端： 主要用于处理服务的注册与发现。向注册中心注册自身提供的服务并周期性的发送心跳来更新服务租约。同时从服务端查询当前注册的服务信息并缓存到本地且周期性的刷新服务。

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


### Reference
1. [SpringCloud教程第1篇：Eureka（F版本）](https://www.fangzhipeng.com/springcloud/2018/08/30/sc-f1-eureka/)

eureka.client.registerWithEureka=false
eureka.client.fetchRegistry=false
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
```

#### 创建SpringCloudClient
1. 在SpringBootClient的启动类中，添加@EnableEurekaClient
```Java
@SpringBootApplication
@EnableEurekaClient
public class SpringCloudClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudClientApplication.class, args);
	}
}
```

2. 定义properties文件
```Properties
server.port=8762
spring.application.name=service-hi # 定义微服务的名字
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
```

### 结果
![Spring Cloud](https://i.imgur.com/UVakJZt.png)
在Server的页面中可以找到Service-hi，说明已经成功注册服务了。


### Reference
1. [SpringCloud教程第1篇：Eureka（F版本）](https://www.fangzhipeng.com/springcloud/2018/08/30/sc-f1-eureka/)
