---
layout: post
title:  "Spring Cloud Hystrix"
date:   2018-11-07 08:43:59
author: Botao Xiao
comment: true
categories: SpringCloud
---
### 雪崩效应
在微服务中，每个微服务都是通过RPC进行交互的，根据前两章中我们可以通过Ribbon，RestTemplate和Feign进行调用。

网络充满了许多不确定的因素，例如网络原因导致了某个微服务不可用，大量的请求涌入（或是别的微服务调用），会迅速消耗完Servlet容器的线程资源。造成服务器宕机。

微服务之间的相互调用会传递雪崩效应。

### 断路器
当对特定的服务的调用的不可用达到一个阀值（Hystric 是5秒20次） 断路器将会被打开。断路打开后，可用避免连锁故障，fallback方法可以直接返回一个固定值。
![断路器模式](https://i.imgur.com/EBTOrSx.png)

我们通过netflix的hystrix实现断路器。我们要通过集成Ribbon和hystrix或Feign和hystrix实现对无法回应的请求的阻塞。
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

### Ribbon中使用断路器
1. 使用@EnableHystrix开启Hystrix断路器
```Java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@EnableHystrix	//开启Hystrix断路器
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
```

2. 断路器适用于Service层，当Ribbon通过RestTemplate进行负载均衡时，如果Ribbon无法进行转发时，就会返回调用Fallback方法。
```Java
@Service
public class HelloService {
	@Autowired
	private RestTemplate restTemplate;
	@HystrixCommand(fallbackMethod="hiError")	//如果调用服务，就会调用hiError方法。并且会将形参进行传递。
	public String hiService(String name){
		return restTemplate.getForObject("http://SERVICE-HI/hi?name=" + name, String.class);
	}
	public String hiError(String name){
		return "Sorry " + name + ", Error happened!";
	}
}
```

### Feign中使用Hystrix
Finchley版本的Spring Cloud中Hystrix没有打开，需要在配置文件中打开Hystrix
```Properties
feign.hystrix.enabled=true
```

1. 在负载均衡的注解（@FeignClient）中加入FallBack所实行的类。
```Java
//定义一个feign接口，通过@ FeignClient（“服务名”），来指定调用哪个服务。
//fallback定义了断路器的默认方法。
@FeignClient(value="SERVICE-HI", fallback=HelloServiceHystrix.class)
public interface HelloService {
	@RequestMapping(value="/hi")
	public String sayHello(@RequestParam("name") String name);
}
//实现一个类继承HelloService，HelloService是默认的负载均衡的方法。
//必须将该类注册到Spring容器中，这样就能通过反射调用到默认的断路方法。
@Component
public class HelloServiceHystrix implements HelloService {
	@Override
	public String sayHello(String name) {
		return "Hello" + name + ", Error happend!";
	}
}
```

### Result
1. 当SERVICE-HI是UP时：
	* Hello sean, return from port: 8766
2. 当SERVICE-HI无法调用时：
	* Sorry sean, Error happened!

### Reference
1. [SpringCloud教程第4篇：Hystrix（F版本）](https://www.fangzhipeng.com/springcloud/2018/08/30/sc-f4-hystrix/)
