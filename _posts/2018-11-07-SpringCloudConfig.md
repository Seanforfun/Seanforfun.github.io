---
layout: post
title:  "Spring Cloud | Spring Cloud Config 配置文件"
date:   2018-11-07 08:43:59
author: Botao Xiao
comment: true
categories: SpringCloud
---
### 配置文件的意义
SpringBoot已经大大简化了配置文件的数量，但是有的服务仍然无法避免配置文件的存在。而微服务的存在使得配置文件数量大大增加，所以我们要使用分布式配置中心组件。

我们可以将配置文件注册在本地或是Git上。

在spring cloud config 组件中，分两个角色，一是config server，二是config client。

### 注意
在git上，我们配置文件的名字是config-client-dev.properties。所以我们在config-client中定义的服务名就应该是config-client，我们应该使用spring.cloud.config.profile=dev来配置文件名的最后一个位置。此点需要注意。


### 创建config-server
1. 创建一个SpringBoot项目，引入config-server模块。
2. 引入@EnableConfigServer开启配置服务器功能。
```Java
@SpringBootApplication
@EnableEurekaClient	//注册服务
@EnableDiscoveryClient
@EnableConfigServer	//注册成为Config-server
public class SpringCloudConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudConfigServerApplication.class, args);
	}
}
```

3. 在配置文件中配置端口和配置文件存储位置。对于公开项目，我们不需要填写用户名和密码。
```Properties
spring.application.name=config-server
server.port=8888	# 默认为8888
spring.cloud.config.server.git.uri=https://github.com/Seanforfun/SpringCloudConfig
spring.cloud.config.server.git.searchPaths=/
spring.cloud.config.label=master
spring.cloud.config.server.git.username=
spring.cloud.config.server.git.password=
```

4. 在仓库中添加配置文件
[config-client-dev.properties](https://github.com/Seanforfun/SpringCloudConfig/blob/master/config-client-dev.properties)

5. http请求地址和资源文件映射如下:
* /{application}/{profile}[/{label}]
* /{application}-{profile}.yml
* /{label}/{application}-{profile}.yml
* /{application}-{profile}.properties
* /{label}/{application}-{profile}.properties


#### 结果
访问http://localhost:8888/config-client/dev，会返回
```xml
<Environment>
<name>config-client</name>
<profiles>
<profiles>dev</profiles>
</profiles>
<label/>
<version>307ed6395ea4ae2e0d01bff274d8f76379092fe4</version>
<state/>
<propertySources>
<propertySources>
<name>
https://github.com/Seanforfun/SpringCloudConfig//config-client-dev.properties
</name>
<source>
<foo>foo version 3</foo>
</source>
</propertySources>
</propertySources>
</Environment>
```

### 创建ConfigClient
1. 创建一个SpringBoot项目作为Client,同时将其注册为一个服务。

```Java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class SpringCloudConfigClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudConfigClientApplication.class, args);
	}
}
```

2. 定义一个properties文件，此处使用bootstrap.properties，因为bootstrap.properties文件优先于application.properties启动。
```Properties
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
spring.application.name=config-client
spring.cloud.config.label=master
spring.cloud.config.profile=dev
spring.cloud.config.uri= http://localhost:8888/	#配置中心的路径
server.port=8881
```

3. 通过@Value注解注入配置
```Java
@RestController
public class HiController {
	@Value("${foo}")
	String foo;
	@RequestMapping("/hi")
	public String sayHi(){
		return foo;
	}
}
```

### Result
http://localhost:8881/hi
foo version 3

### Reference
1. [SpringCloud教程第6篇：config（F版本）](https://www.fangzhipeng.com/springcloud/2018/08/30/sc-f6-config/)
