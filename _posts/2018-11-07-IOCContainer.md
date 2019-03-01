---
layout: post
title:  "Spring | IOC Container"
date:   2018-11-07 19:43:59
author: Botao Xiao
categories: Spring
comment: true
description: Spring容器的总结和应用。
---
# IOC Container

### IOC容器的初始化
![IOC容器的初始化](https://i.imgur.com/shMORhc.jpg)
```Java
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		// Prepare this context for refreshing.
		prepareRefresh();
		// Tell the subclass to refresh the internal bean factory.
		// 初始化BeanFactory
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
		// Prepare the bean factory for use in this context.
		prepareBeanFactory(beanFactory);
		try {
			// Allows post-processing of the bean factory in context subclasses.
			postProcessBeanFactory(beanFactory);
			// Invoke factory processors registered as beans in the context.
			//调用工厂后处理器
			invokeBeanFactoryPostProcessors(beanFactory);
			// Register bean processors that intercept bean creation.
			//注册Bean后处理器
			registerBeanPostProcessors(beanFactory);
			// 初始化消息源
			initMessageSource();
			// 初始化应用上下文时间广播器
			initApplicationEventMulticaster();
			// Initialize other special beans in specific context subclasses.
			// 初始化其他特殊的Bean, 具体由子类实现。
			onRefresh();
			// 注册监听器事件
			registerListeners();
			// 初始化所有的bean（不包括懒加载）
			finishBeanFactoryInitialization(beanFactory);
			// 完成刷新并发布容器刷新事件
			finishRefresh();
		}
		catch (BeansException ex) {
			//...
		}
	}
}
```

#### BeanDefinition
> BeanDefinition是一种对xml的java代码解释，和xml中的一个<bean>对应，在<bean>标签中的属性和BeanDefinition中的属性相对应。

1. 通过org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(RootBeanDefinition, String, BeanFactory)方法生成对象的实例
```Java
public Object instantiate(RootBeanDefinition bd, String beanName, BeanFactory owner) {
	// Don't override the class with CGLIB if no overrides.
	if (bd.getMethodOverrides().isEmpty()) {
		Constructor<?> constructorToUse;
		synchronized (bd.constructorArgumentLock) {
			constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;	//获取构造器或工厂方法。
			if (constructorToUse == null) {
				final Class<?> clazz = bd.getBeanClass();	//获得类对象
				if (clazz.isInterface()) {
					throw new BeanInstantiationException(clazz, "Specified class is an interface");	//如果当前的类是一个接口则无法被实例化。
				}
				try {
					//检查SecurityManager以获取可以使用的构造器方法。
					if (System.getSecurityManager() != null) {
						constructorToUse = AccessController.doPrivileged(new PrivilegedExceptionAction<Constructor<?>>() {
							@Override
							public Constructor<?> run() throws Exception {
								return clazz.getDeclaredConstructor((Class[]) null);
							}
						});
					}
					else {
						constructorToUse = clazz.getDeclaredConstructor((Class[]) null);
					}
					bd.resolvedConstructorOrFactoryMethod = constructorToUse;
				}
				catch (Exception ex) {
					throw new BeanInstantiationException(clazz, "No default constructor found", ex);
				}
			}
		}
		//根据构造器获取一个对象实例。
		return BeanUtils.instantiateClass(constructorToUse);
	}
	else {
		// Must generate CGLIB subclass.
		return instantiateWithMethodInjection(bd, beanName, owner);
	}
}
```

#### BeanWrapper
1. Bean包裹器
2. 属性访问器
3. 属性编辑器注册表

#### 使用properties文件
1. 在xml引入context命名空间，并定义properties文件的路径
```xml
<!-- 通过context域引入properties文件 -->
<context:property-placeholder location="classpath:jdbc.properties"/>
<!-- 通过PropertyPlaceholderConfigurer引入properties文件 -->
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer" scope="singleton">
	<property name="location" value="classpath:jdbc.properties"/>
</bean>
```

2. 将properties文件存储的内容进行注入
```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
	<property name="driverClassName" value="${driverClassName}"/>
	<property name="url" value="${url}"/>
	<property name="username" value="${username}"/>
	<property name="password" value="${password}"/>
</bean>
```

3. 通过注解引入properties文件
* 原始类
```Java
@Component("myDataSource")
public class MyDataSource {
	@Value("${driverClassName}")	//通过改种方法将属性的值进行值的注入。
	private String driverClassName;
	@Value("${url}")
	private String url;
	@Value("${username}")
	private String username;
	@Value("${password}")
	private String password;
	public String getDriverClassName() {
		return driverClassName;
	}
	public String getUrl() {
		return url;
	}
	public String getUsername() {
		return username;
	}
	public String getPassword() {
		return password;
	}
}
```

4. Properties文件自身的引用
* properties文件可以将内容分开存储相互引用。
```Properties
dbName=sampledb
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/${dbName} #此处的dbName引用了别处定义的值。
```

### 引用Bean的属性值
> 有的配置信息希望放在bean内部由代码动态控制，所以我们有时候希望引用新的bean的值动态调整系统的信息。

* 通过xml获得bean对象的传入值
```xml
<bean id="dataSourceConfig" class="ca.mcmaster.spring.di.DataSourceConfig">
	<!-- 通过已经定义的bean对象获得传入的值 -->
    <property name="maxTimeOut" value="#{sysConfig.maxTimeOut}"/>
    <property name="connections" value="#{sysConfig.connections}"/>
</bean>
```

* 通过注解获得bean对象传入的值
```Java
@Component("dataSourceConfig")
public class DataSourceConfig {
	@Value("#{sysConfig.maxTimeOut}")	//和xml式的注入很类似。
	private Integer maxTimeOut;
	@Value("#{sysConfig.connections}")
	private Integer connections;
	public Integer getMaxTimeOut() {
		return maxTimeOut;
	}
	public Integer getConnections() {
		return connections;
	}
}
```

### 国际化
1. 传统的国际化，通过NumberFormat实现单位的国际化。
```Java
Locale ch = new Locale("zh", "CN");
NumberFormat zhnf = NumberFormat.getCurrencyInstance(ch);
Locale ca = new Locale("en", "CA");
NumberFormat canf = NumberFormat.getCurrencyInstance(ca);
double currency = 12345.6D;
System.out.println(zhnf.format(currency)); //￥12,345.60
System.out.println(canf.format(currency)); //$12,345.60
```

2. 对于Message进行国际化装配
```Java
// 信息格式化串
String chMsg = "{0}, 你好！你于{1}在工商银行存入{2}元！";
String usMsg = "At {1, time, short} On {1, date,long}, {0} saved {2, number, currency}";
// 定义动态占位符的参数
Object[] params = {"Sean", new GregorianCalendar().getTime(), 1000000000};
// 指定国际化信息
MessageFormat zhMf = new MessageFormat(chMsg, Locale.CHINESE);
//装配参数
String zhPattern = zhMf.format(params);
System.out.println(zhPattern);	//Sean, 你好！你于18-7-19 上午9:59在工商银行存入1,000,000,000元！
MessageFormat usMf = new MessageFormat(usMsg, Locale.CANADA);
String usPattern = usMf.format(params);
System.out.println(usPattern);	//At 9:59 AM On July 19, 2018, Sean saved $1,000,000,000.00
```

3. 通过ResourceBundle实现国际化
* 在项目中定义国际化的properties文件，该文件的文件名有一定的规范：<资源名>_<语言名>.properties， 例如greeting_en_CA.properties和greeting_zh_CN.properties。
* 在代码中调用
```Java
ResourceBundle enBundle = ResourceBundle.getBundle("ca/mcmaster/spring/i18n/greeting", Locale.CANADA);	//前一个参数写出了全路径名（最后一个位置为资源名），第二个参数是国际化的信息。
ResourceBundle cnBundle = ResourceBundle.getBundle("ca/mcmaster/spring/i18n/greeting", Locale.CHINESE);
System.out.println(enBundle.getString("greeting.common"));
System.out.println(new String(cnBundle.getString("greeting.common").getBytes("ISO-8859-1"), "utf-8"));	//properties文件是通过ISO-8859-1编码集编码的，我们要转到utf-8编码集。
Hello!
ä½ å¥½ï¼	//未转编码集的输出结果。
你好！
```

4. 资源文件中使用格式化串
* 在资源文件中定义语句串
* 在Java文件中获取语句串并赋值
```Java
greeting.common=Hello {0}! Today is {1}!
ResourceBundle fmtBundle = ResourceBundle.getBundle("ca/mcmaster/spring/i18n/fmt_greeting", Locale.CANADA);
MessageFormat msgFormat = new MessageFormat(fmtBundle.getString("greeting.common"), Locale.CANADA);
Object[] params = {"Sean", new GregorianCalendar().getTime()};
String formatMsg = msgFormat.format(params);
System.out.println(formatMsg);
Hello Sean! Today is 19/07/18 10:34 AM!
```

5. 利用Spring来获取ResourceBundle信息
	* 可以通过ReloadableResourceBundleMessageSource来设置cacheSeconds使框架会刷新resource文件。

```Xml
<bean id="myResource" class="org.springframework.context.support.ResourceBundleMessageSource">
	<property name="basenames">
		<list>
    		<value>ca/mcmaster/spring/i18n/fmt_greeting</value>
    	</list>
    </property>
</bean>
```

### 容器事件
> Spring的ApplicationContext能够发布事件并且允许注册相应的事件监听器，它拥有一套完善的事件发布和监听机制。

#### 事件体系（实际上使用的即是观察者模式）
* 事件源：事件的生产值，任何一个事件都要有一个事件源。
* 监听器注册表: 用于存储事件监听器。
* 事件广播器：用于将事件发布给事件监听器。

![事件体系](https://i.imgur.com/DwxGqVW.png)

#### Spring事件结构
![Spring事件结构](https://i.imgur.com/9IucCjs.png)
1. 和容器相关的事件继承了ApplicationContextEvent,并定义了容器的关闭，刷新，启动和停止事件。
2. 和网络相关的事件需要在web.xml中定义DispatcherServlet，分别代表了servlet和Portlet请求事件。

#### Spring事件监听接口
![Spring事件监听接口](https://i.imgur.com/SMWXxSQ.png)
1. 实现onApplicationEvent(E event)方法，事件进行相应。
2. supportsEventType(Class<? extends ApplicationEvent> eventType)：定义何种事件可以被响应。
3. supportsSourceType(Class<?> sourceType)：什么样的消息源可以被响应。

#### 事件广播器
![事件广播器](https://i.imgur.com/eANohIs.png)

1. 容器主控程序会通过事件广播器将事件通知给注册表中的事件监听器，监听器会对事件分别进行相应。

#### Spring事件体系的实现
1. 初始化事件广播器initApplicationEventMulticaster
```Java
protected void initApplicationEventMulticaster() {
	ConfigurableListableBeanFactory beanFactory = getBeanFactory();
	//如果当前的beanFactory已经存在了一个广播器则获取当前的广播器。
	if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
		this.applicationEventMulticaster =
				beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
		if (logger.isDebugEnabled()) {
			logger.debug("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
		}
	}
	else {
	//如果当前beanfactory还没有广播器则生成并注册。
		this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
		beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
		if (logger.isDebugEnabled()) {
			logger.debug("Unable to locate ApplicationEventMulticaster with name '" +
					APPLICATION_EVENT_MULTICASTER_BEAN_NAME +
					"': using default [" + this.applicationEventMulticaster + "]");
		}
	}
}
```

2. 注册事件监听器registerListeners
```Java
protected void registerListeners() {
	// Register statically specified listeners first.
	for (ApplicationListener<?> listener : getApplicationListeners()) {
		// 将所有的事件接听器注册到事件广播器中
		getApplicationEventMulticaster().addApplicationListener(listener);
	}
	// Do not initialize FactoryBeans here: We need to leave all regular beans
	// uninitialized to let post-processors apply to them!
	// 通过反射将所有的ApplicationListener的bean名称并将这些bean对象加入广播器中。
	String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
	for (String lisName : listenerBeanNames) {
		getApplicationEventMulticaster().addApplicationListenerBean(lisName);
	}
}
```

3. 发布事件publishEvent
```Java
public void publishEvent(ApplicationEvent event) {
	Assert.notNull(event, "Event must not be null");
	if (logger.isTraceEnabled()) {
		logger.trace("Publishing event in " + getDisplayName() + ": " + event);
	}
	getApplicationEventMulticaster().multicastEvent(event);	//将事件发布
	if (this.parent != null) {
		this.parent.publishEvent(event);
	}
}
```

4. 事件发布的实例
* 定义事件
```Java
public class MailSendEvent extends ApplicationContextEvent {
	private String to;
	public MailSendEvent(ApplicationContext source, String to) {
		super(source);
		this.to = to;
	}
	public String getTo() {
		return to;
	}
}
```

* 事件监听器
```Java
public class MailSendListener implements ApplicationListener<MailSendEvent> {
	/* (non-Javadoc)
	 * @see org.springframework.context.ApplicationListener#onApplicationEvent(org.springframework.context.ApplicationEvent)
	 * 对事件进行处理。即是事件响应的处理事件。
	 */
	@Override
	public void onApplicationEvent(MailSendEvent event) {
		System.out.println("MailSendListener: Send mail to " + event.getTo() + ".");
	}
}
```

* 事件的发布
> 只有实现了ApplicationContextAware才会被spring调用，将当前的bean容器加入bean对象使当前对象调用上下文信息。

```Java
@Component("mailSender")
public class MailSender implements ApplicationContextAware {
	private ApplicationContext applicationContext;
	@Override
	public void setApplicationContext(ApplicationContext applicationContext)
			throws BeansException {
		//将容器注册
		this.applicationContext = applicationContext;
	}
	public void sendMail(String to){
		System.out.println("MailSender: Send a mail!");
		//创建一个发送事件
		MailSendEvent mailSendEvent = new MailSendEvent(this.applicationContext, to);
		//发布事件
		applicationContext.publishEvent(mailSendEvent);
	}
}
```

* 事件的调用
```Java
mailSender.sendMail("xiaob6@mcmaster.ca");	//发送过程中会创建事件，并将事件发布。
```

* 而Spring框架会调用multicastEvent方法，调用线程池，开启线程执行响应
```Java
	public void multicastEvent(final ApplicationEvent event) {
		for (final ApplicationListener<?> listener : getApplicationListeners(event)) {
			Executor executor = getTaskExecutor();
			if (executor != null) {
				executor.execute(new Runnable() {
					@Override
					public void run() {
						invokeListener(listener, event);
					}
				});
			}
			else {
				invokeListener(listener, event);
			}
		}
	}
	protected void invokeListener(ApplicationListener listener, ApplicationEvent event) {
		ErrorHandler errorHandler = getErrorHandler();
		if (errorHandler != null) {
			try {	//调用监听器的响应方法
				listener.onApplicationEvent(event);
			}
			catch (Throwable err) {
				errorHandler.handleError(err);
			}
		}
		else {
			listener.onApplicationEvent(event);
		}
	}
```
