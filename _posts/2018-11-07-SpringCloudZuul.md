---
layout: post
title:  "Spring Cloud Zuul 路由网关"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: SpringCloud
description: 在微服务架构中，需要几个基础的服务治理组件，包括服务注册与发现、服务消费、负载均衡、断路器、智能路由、配置管理等，由这几个基础组件相互协作，共同组建了一个简单的微服务系统。
---
### 微服务构架
![微服务构架](https://i.imgur.com/COoCyXU.png)
在微服务架构中，需要几个基础的服务治理组件，包括服务注册与发现、服务消费、负载均衡、断路器、智能路由、配置管理等，由这几个基础组件相互协作，共同组建了一个简单的微服务系统。

### Zuul
Zuul实现了路由器和过滤器功能。Zuul默认和Ribbon实现了负载均衡。
* Authentication 权限校验
* Insights
* Stress Testing 压力测试
* Canary Testing
* Dynamic Routing 智能路由
* Service Migration
* Load Shedding
* Security
* Static Response handling
* Active/Active traffic management

### Zuul的网关
#### Zuul网关配置流程
1. 创建一个SpringBoot项目，添加Zuul模块。并通过@EnableZuulProxy开启Zuul功能。
```Java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@EnableZuulProxy	//开启Zuul路由网关功能
public class SpringCloudZuulApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudZuulApplication.class, args);
	}
}
```

2. 在配置文件中定义网关。
```Properties
server.port=8768
spring.application.name=service-zuul
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
# Define gateway to ribbon
zuul.routes.ribbon.path=/ribbon/**
zuul.routes.ribbon.serviceId=service-ribbon
# Define gateway to Feign
zuul.routes.feign.path=/feign/**
zuul.routes.feign.serviceId=service-feign
```

#### 结果
http://localhost:8768/feign/hi?name=sean
Hello sean, return from port: 8766
http://localhost:8768/ribbon/hi?name=sean
Hello sean, return from port: 8766
流程：
1. Zuul服务接收到RESTFul指令，解析指令。
2. 通过path进行匹配，决定解决该服务的服务名。
3. 通过Ribbon+RestTemplate或Feign进行RPC，SERVICE-HI服务执行业务。

### 服务过滤
定义一个过滤器,继承Zuul的ZuulFilter类，用于进行服务校验和匹配。
* filterType， 决定什么时候进行匹配拦截：
	* pre：路由之前
	* routing：路由之时
	* post： 路由之后
	* error：发送错误调用

```Java
@Component
public class MyFilter extends ZuulFilter {
	private static Logger logger = LoggerFactory.getLogger(MyFilter.class);
	@Override
	public Object run() throws ZuulException {
		//通过Zuul的Context环境获取parameter
		RequestContext ctx = RequestContext.getCurrentContext();
		//从request域中获取parameter进行判断。
		String token = ctx.getRequest().getParameter("token");
		if(token == null){
			logger.warn("token is empty!");
			// 未通过权限校验，Zuul将不会把服务进行转发。
			ctx.setSendZuulResponse(false);
			// 设置response的返回状态值
			ctx.setResponseStatusCode(401);
			try {
				ctx.getResponse().getWriter().write("Token is empty!");
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		logger.info("Pass!");
		return null;
	}
	//什么时候进行过滤
	@Override
	public boolean shouldFilter() {
		return true;
	}
	//决定在什么时候进行拦截
	@Override
	public String filterType() {
		return "pre";
	}
	//filterOrder：过滤的顺序
	@Override
	public int filterOrder() {
		return 0;
	}
}
```

#### 结果
http://localhost:8768/ribbon/hi?name=sean&token=121
Hello sean, return from port: 8766
http://localhost:8768/ribbon/hi?name=sean
Token is empty!

### Reference
1. [SpringCloud教程第5篇：Zuul（F版本）](https://www.fangzhipeng.com/springcloud/2018/08/30/sc-f5-zuul/)