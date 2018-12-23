---
layout: post
title:  "Spring Cloud Ribbon"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: SpringCloud
comment: true
description: Ribbon是一个Client端的负载均衡器。Ribbon可以控制Http和TCP的行为，实现在微服务端的负载均衡。在同一个服务多次启动的时候，会形成一个集群，而Ribbon的作用就是使服务在集群中负载均衡。
---
### Load balance
Load balancer分为两种，一种是通过硬件进行负载均衡，另一种是通过软件进行负载均衡，然而两种实现方法的模式均可用下图表示。下图是通过五元组进行负载均衡的实现方法。
![Imgur](https://i.imgur.com/qued2o9.png)

### 负载均衡的模型
负载均衡又分为四层负载均衡和七层负载均衡。四层负载均衡工作在OSI模型的传输层，主要工作是转发，它在接收到客户端的流量以后通过修改数据包的地址信息将流量转发到应用服务器。在传输层，我们可以获取五元组信息，我们则通过我们的负载均衡算法，修改报文的请求的ip地址。这样的工作是在传输层就可以执行，此时已经拥有了五元组信息。

七层负载均衡工作在OSI模型的应用层，因为它需要解析应用层流量，所以七层负载均衡在接到客户端的流量以后，还需要一个完整的TCP/IP协议栈。七层负载均衡会与客户端建立一条完整的连接并将应用层的请求流量解析出来，再按照调度算法选择一个应用服务器，并与应用服务器建立另外一条连接将请求发送过去，因此七层负载均衡的主要工作就是代理。这意味代理服务器同时建立了和客户端的TCP链接（不然无法获得应用层信息）和服务器的TCP连接。

![TCP建立连接，三次握手](https://i.imgur.com/epurUYW.jpg)

在这种情况下，服务器端并不对外界公开集群中的各个节点的ip地址，而负载均衡器本身是公开的，就像代理服务器一样，负载均衡器可以根据负载均衡算法分配哪个结点对当前的请求进行处理。

### Ribbon
Ribbon是一个Client端的负载均衡器。Ribbon可以控制Http和TCP的行为，实现在微服务端的负载均衡。在同一个服务多次启动的时候，会形成一个集群，而Ribbon的作用就是使服务在集群中负载均衡。

通过Ribbon实现负载均衡的步骤：
* 服务提供多个服务实例并注册到一个（或多个相关联的）注册中心，原则就是保证这些服务是可见的。
* 服务消费者通过调用@LoadBalanced注解修饰过得RestTemplate来实现面向服务的接口调用。

![多个微服务形成集群](https://i.imgur.com/8ujJBVQ.png)

### Ribbon对RESTFul api的支持： RestTemplate
1. GET： 请求获取资源，不修改当前的所有资源。
    * <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables) throws RestClientException;
    * public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException
    * responseType是用作定义返回值的对象类型的，通过uriVariables传递参数。
    * demo
    ```Java
    @RequestMapping(value = "/ribbon/{name}/{id}", method = RequestMethod.GET)
    public String getUser(@PathVariable("name") String name, @PathVariable("id") Integer id){
        Map<String, Object> map = new HashMap<>();  // 从restAPI的路径中获取参数，存入map
        map.put("name", name);
        map.put("id", id);
        ResponseEntity<String> entity = restTemplate.getForEntity("http://hello-service/hello/{name}/{id}", String.class, map);   //将参数分装好后进行传递。
        return entity.getBody();
    }
    @RequestMapping(value = "/hello/{name}/{id}", method = RequestMethod.GET)
    public String getUser(@PathVariable("name") String name, @PathVariable("id") Integer id){
        log.info("name is {} id is {}", name, id);
        User user = new User(name, id);
        return JSONObject.toJSONString(user);   //将对象封装成Json并返回字符串。
    }
    ```

2. POST： 用于创建新的资源（也可用于修改）
    * public <T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException， 其中request是要发布的对象。
    * public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException
    * public URI postForLocation(String url, @Nullable Object request, Map<String, ?> uriVariables) throws RestClientException //返回值是URI，代表的是新资源的路径。

3. Put: 用于修改资源
    * public void put(String url, @Nullable Object request, Map<String, ?> uriVariables)
			throws RestClientException

4. Delete: 用于删除资源
    * public void delete(String url, Map<String, ?> uriVariables) throws RestClientException

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
2. [负载均衡基础知识](https://www.cnblogs.com/danbing/p/7459224.html)
