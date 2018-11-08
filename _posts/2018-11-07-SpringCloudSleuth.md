---
layout: post
title:  "Spring Cloud Sleuth 服务追踪"
date:   2018-11-07 08:43:59
author: Botao Xiao
comment: true
categories: SpringCloud
---
### 服务追踪
所有的业务都是由多个微服务构成的，随着业务的扩大，单个业务将会由很多的的微服务实现，这就意味着，当某个业务出现了问题，我们将会很难知道该业务从哪儿来，到哪儿去，是哪个业务出了问题，造成了缓慢。此时我们就需要服务追踪。

### 术语
1. Span(跨度)： Span服务调用的基本单元，我个人的理解是，span实际上是一次RPC的调用，每一次的请求就是一个Span。
2. Trace（痕迹）：Trace是由Span组成的，所有的Span将所有的结点连接起来，形成了一张调用图。
3. 标注（Annotation）：用于及时记录事件的存在。用于定义请求开始和结束的一些核心注释是：
	* cs - 客户端发送 - 客户端发出请求。这个注释描述了跨度的开始。
	* sr - 服务器已收到 - 服务器端收到请求并开始处理。如果从这个时间戳中减去cs时间戳，将会收到网络延迟。
	* ss - 服务器发送 - 在请求处理完成时（当响应被发送回客户端时）注释。如果从这个时间戳中减去sr时间戳，将会收到服务器端处理请求所需的时间。
	* cr - 客户端收到 - 表示跨度结束。客户端已经成功接收到服务器端的响应。如果从这个时间戳中减去cs时间戳，那么将会收到客户端接收服务器响应所需的全部时间。

### 流程
1. 打开Zipkin的服务器程序。通过cmd打开。
2. 实现两个相互调用的微服务。
3. 通过访问http://http://localhost:9411 来找到调用流程。可以发现服务之间的相互调用关系。

![服务追踪](https://i.imgur.com/prlIkKG.png)

### Reference
1. [SpringCloud教程第8篇：Spring Cloud Bus（F版本）](https://www.fangzhipeng.com/springcloud/2018/08/30/sc-f8-bus/)
2. [Spring Cloud Sleuth和zipkin微服务跟踪](https://blog.csdn.net/qq_15144655/article/details/80020199)
