---
layout: post
title:  "Spring Cloud注册与发现"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: SpringCloud
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
eureka.instance.hostname=Seanforfun
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