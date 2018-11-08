---
layout: post
title:  "Listener"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: JavaBackend
description: Listener是用于监听Servlet行为的工具。对于属性变化或者环境变化进行监控的器件。
---
## Listener

### Listener
Listener是用于监听Servlet行为的工具。
对于属性变化或者环境变化进行监控的器件。
参考[观察者模式](https://github.com/Seanforfun/JavaCore/blob/master/Conclusions/Observer.md)

#### Listener 的实现模式
* 我们通过重写监听器中的方法，在环境发生变化的时候，会调用观察者，对每一个订阅的对象进行更新。

1. 对于servlet生命周期的监控。
	* ServletContextListener
		* contextInitialized 在容器启动时被调用（在servlet被实例化前执行）
		* contextDestroyed 在容器销毁时调用（在servlet被销毁后执行）
	* HttpSessionListener
		* sessionCreated 在HttpSession创建后调用
		* sessionDestroyed 在HttpSession销毁前调用（执行session.invalidate();方法）
	* ServletRequestListener
		* requestDestroyed 在request对象创建后调用（发起请求）
		* requestInitialized 在request对象销毁前调用（请求结束）
2. 属性变化监听器
	* ServletRequestAttributeListener
		* attributeAdded(ServletContextAttributeEvent event)　　向appliction中添加属性时调用
		* attributeRemoved(ServletContextAttributeEvent event)　　从appliction中删除属性时调用
		* attributeReplaced(ServletContextAttributeEvent event)　　替换application中的属性时调用
	* HttpSessionAttributeListener
	* ServletRequestAttributeListener
3. session中指定类属性变化监听器
	* HttpSessionBindingListener
		* valueBound(HttpSessionBindingEvent event) 当该类实例设置进session域中时调用
		* valueUnbound(HttpSessionBindingEvent event) 当该类的实例从session域中移除时调用
	* HttpSessionActivationListener
		* sessionWillPassivate(HttpSessionEvent se) 当对象session被序列化（钝化）后调用
		* sessionDidActivate(HttpSessionEvent se)  当对象session被反序列化（活化）后调用

### Listener的配置
* 通过xml配置

```xml
<listener>
	<listener-class>study.myListener.ServletContentAttribute_Listener</listener-class>
</listener>
```

* 通过注解配置

```Java
@WebListener
```


### Reference
1. [Listener监听器生命周期](https://www.cnblogs.com/caijh/p/7683007.html)