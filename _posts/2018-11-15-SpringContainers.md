---
layout: post
title:  "Spring| IOC Container: BeanFactory, ApplicationContext"
date:   2018-11-15 15:49
author: Botao Xiao
categories: Spring
comment: true
description: Spring中的IOC容器描述了Bean和Bean之间的关系，并且在此基础上提供了Bean实例缓存，生命周期管理，Bean实例代理，事件发布，资源装载等高级功能。具体的实现类是BeanFactory和ApplicationContext，前者是面向Spring框架本身的，而ApplicationContext是面向程序员的。
---
Spring中的IOC容器描述了Bean和Bean之间的关系，并且在此基础上提供了Bean实例缓存，生命周期管理，Bean实例代理，事件发布，资源装载等高级功能。具体的实现类是BeanFactory和ApplicationContext，前者是面向Spring框架本身的，而ApplicationContext是面向程序员的。

### BeanFactory
1. BeanFactory是通用的类工厂，它可以创建并管理各种类对象。
2. BeanFactory的源码
```Java
public interface BeanFactory {
    // 获取Bean的多种方式
	Object getBean(String name) throws BeansException;
	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
	Object getBean(String name, Object... args) throws BeansException;
	<T> T getBean(Class<T> requiredType) throws BeansException;
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
	// 获取对象的工厂方法
	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
	boolean containsBean(String name);
	// 确定对象是单例模式还是原型模式
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
	// 获取某个对象的类名
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	String[] getAliases(String name);
}
```

### Application Context
1. ApplicationContext是由BeanFactory派生出来的，更多的是面向程序员，实际上在编程的层面，我们只需要ApplicationContext即可。
2. ApplicationContext的源码
```Java
/**
 *ApplicationEventPublisher: 让容器拥有发布上下文事件的功能
 *MessageSource：提供i18n的信息
 *ResourcePatternResolver:资源文件解析器，获取资源文件。
 */
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
	@Nullable
	// 获取上下文中的bean的信息
	String getId();
	String getApplicationName();
	String getDisplayName();
	long getStartupDate();
	@Nullable
	ApplicationContext getParent();
	AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```
* ApplicationEventPublisher: 让容器拥有发布上下文事件的功能，使用了[观察者模式](http://localhost:4000/designpattern/2018/11/07/Observer.html)。可以发布启动事件和关闭事件等等。
* MessageSource：实现i18n的信息，将国际化的信息取出，并进行填充。使用了[策略模式](http://localhost:4000/designpattern/2018/11/07/Strategy.html)。
* ResourcePatternResolver：仍然使用策略模式，获取资源文件，其中使用了[Spring的资源获取方法](https://seanforfun.github.io/spring/2018/11/14/SpringResource.html)。
* LifeCycle: 策略模式，一般ApplicationContext的实现类都会实现该接口，用于控制异步处理的过程。

3. 使用注解定义新的bean对象。
```Java
@Slf4j
@ToString
@Component  //使用@Component对象，让该类能被Spring容器发现。
public class Car {
    @Setter
    @Getter
    private Integer id;
    @Setter
    @Getter
    private String name;
    @Setter
    @Getter
    private Integer maxSpeed;
    @Bean(name = "myCar")
    @Scope("singleton")
    public Car buildCar(){
        Car car = new Car();
        car.setId(1);
        car.setMaxSpeed(200);
        car.setName("Toyota");
        return car;
    }
}
```

4. WebApplicationContext
WebApplicationContext允许从Web根目录装载配置文件完成初始化任务。从WebApplicationContext中可以获得Servlet的引用。其中最重要的方法就是获取ServletContext对象。

WebApplicationContext必须要在Web容器启动的前提下才能启动。
```Java
/**
 *Return the standard Servlet API ServletContext for this application.
 */
@Nullable
ServletContext getServletContext();
```

### Bean的生命周期
#### BeanFactory中Bean的生命周期
1. 调用者调用了getBean(beanName)向容器请求一个Bean, 【可选】调用org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation方法。
2. 根据配置调用Bean的构造方法或是工厂方法。
3. 【可选】此时bean实例已经生成，调用org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation方法，对实例进行装扮。
4. 【可选】如果bean设置了属性值，调用org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor#postProcessProperties方法。这一步是对属性值进行加载。
5. 调用bean的属性值设置方法设置属性值。
6. 【可选】调用org.springframework.beans.factory.BeanNameAware#setBeanName方法对bean名称进行设置。
7. 【可选】如果bean实现了org.springframework.beans.factory.BeanFactoryAware#setBeanFactory方法，则将bean工厂注入到bean对象中。
8. 【可选】调用org.springframework.beans.factory.config.BeanPostProcessor#postProcessBeforeInitialization方法，可以改变某些bean的行为，进行前值增强（AOP）。
9. 【可选】调用org.springframework.beans.factory.InitializingBean#afterPropertiesSet方法，设置属性值。
10. 【可选】如果定义了init-method, 将会调用该方法。
11. 【可选】调用org.springframework.beans.factory.config.BeanPostProcessor#postProcessAfterInitialization方法，改变bean的行为，可以进行后置增强。
12. 如果当前bean的作用域是prototype（实际上是设计模式的[原型模式](http://localhost:4000/designpattern/2018/11/07/Prototype.html)）,此时用户将会负责当前bean的生命周期。
13. 如果当前bean是单例模式，会将该单例加入Spring缓存池。Spring将会对Bean后续生命周期进行管理。
14. 【可选】在销毁对象之前，如果注册了org.springframework.beans.factory.DisposableBean#destroy方法，可以在这个方法中释放资源。












