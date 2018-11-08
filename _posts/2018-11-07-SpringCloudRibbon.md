---
layout: post
title:  "Spring Cloud Ribbon"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: SpringCloud
comment: true
description: Ribbon是一个Client端的负载均衡器。Ribbon可以控制Http和TCP的行为，实现在微服务端的负载均衡。在同一个服务多次启动的时候，会形成一个集群，而Ribbon的作用就是使服务在集群中负载均衡。
---
### Ribbon
Ribbon是一个Client端的负载均衡器。

Ribbon可以控制Http和TCP的行为，实现在微服务端的负载均衡。

在同一个服务多次启动的时候，会形成一个集群，而Ribbon的作用就是使服务在集群中负载均衡。

![多个微服务形成集群](https://i.imgur.com/8ujJBVQ.png)

### RIBBON负载均衡的模式
![RIBBON负载均衡的模式](https://i.imgur.com/ASLwA0Z.png)

1. 注册一个Spring cloud server。
![所有注册的微服务](https://i.imgur.com/3cvZ31X.png)
2. 开启两个SERVICE-HI微服务，执行业务。
```Java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@RestController
public class SpringCloudClientApplication {
	@Value("${server.port}")
	private String port;
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudClientApplication.class, args);
	}
	@RequestMapping("/hi") //接收RESTFUL请求。
	private String sayHello(@RequestParam(required=true, value="name") String name){
		return "Hello " + name + ", " + "return from port: " + this.port;
	}
}
```

3. 开启一个新的服务，注入一个RestTemplate对象用于执行负载均衡。
```Java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class SpringCloudClientRibbonApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudClientRibbonApplication.class, args);
	}
	@Bean
	@LoadBalanced
	RestTemplate restTemplate(){
		return new RestTemplate();
	}
}
@Service
public class HelloService {
	@Autowired
	private RestTemplate restTemplate;
	public String hiService(String name){
		//通过负载均衡将REST请求分发到集群。
		return restTemplate.getForObject("http://SERVICE-HI/hi?name=" + name, String.class);
	}
}
```

### 结果
Hello sean, return from port: 8762

Hello sean, return from port: 8766

### Reference
1. [SpringCloud教程第2篇：Ribbon(F版本)](https://www.fangzhipeng.com/springcloud/2018/08/30/sc-f2-ribbon/)
