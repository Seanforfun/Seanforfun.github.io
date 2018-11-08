---
layout: post
title:  "IoC 反转控制"
date:   2018-11-07 19:43:59
author: Botao Xiao
categories: Spring
comment: true
description: 面向对象的处理中，对象封装了数据和堆数据的处理，对象的依赖关系常常体现在对数据和方法的依赖上。所以我们可以将对象之间的依赖交给IoC容器去处理，减少了对象之间的耦合。
---
# IoC 反转控制
> 面向对象的处理中，对象封装了数据和堆数据的处理，对象的依赖关系常常体现在对数据和方法的依赖上。所以我们可以将对象之间的依赖交给IoC容器去处理，减少了对象之间的耦合。

### 核心
1. 反转
2. 控制

### What is Inversed?
1. 在我们原来生成一个对象的实例时，是我们主动将对象的依赖进行注入，例如我们会生成新的实例并赋值给引用，这是正向的控制。
2. 反转控制是指，现在我们生成一个对象的实例，由IoC容器将依赖进行被动注入（注意，此时注入依赖的行为已经不是主动地了，而是需要什么就会被IoC容器注入什么），从而实现了控制的反转。
3. 反转指的是责任的反转，原本依赖由对象主动生成转换成被IoC容器注入。

### 注入的方法
1. 构造函数注入：在构造函数中将属性传入对象。
2. 属性注入：通过get和set方法注入。
3. 接口注入：将调用类的所有依赖抽取到一个接口中，通过实现该接口实现方法的注入。
4. 通过反射注入：
```Java
public class Car {	//此处的Car是一个POJO
	private String color;
	private String brand;
	public String getColor() {
		return color;
	}
	public void setColor(String color) {
		this.color = color;
	}
	public String getBrand() {
		return brand;
	}
	public void setBrand(String brand) {
		this.brand = brand;
	}
}
public class ReflectTest {
	public static Car getCarInstance() throws ClassNotFoundException, Exception, SecurityException{
		ClassLoader loader = Thread.currentThread().getContextClassLoader();	//此处获取的是ApplicationClassLoader.
		Class<?> carClass = loader.loadClass("ca.mcmaster.spring.reflect.Car");
		Constructor<?> constructor = carClass.getConstructor(null);	//获取构造器，并通过构造器创建一个类的对象。
		Car car = (Car) constructor.newInstance();
		Method setColorMethod = carClass.getDeclaredMethod("setColor", String.class);	//通过反射获取方法，通过方法将属性进行注入。
		setColorMethod.invoke(car, "Grey");
		Method setBrandMethod = carClass.getDeclaredMethod("setBrand", String.class);
		setBrandMethod.invoke(car, "RAV4");
		return car;
	}
	public static void main(String[] args) throws ClassNotFoundException, SecurityException, Exception {
		Car car = ReflectTest.getCarInstance();
		System.out.println("Color: " + car.getColor());
		System.out.println("Brand: " + car.getBrand());
	}
}
```

### ClassLoader
```Java
public class ClassloaderTest {
	public static void main(String[] args) {
		ClassLoader applicationClassLoader = Thread.currentThread().getContextClassLoader();
		System.out.println("Current classloader: " + applicationClassLoader);
		System.out.println("Parent classloader: " + applicationClassLoader.getParent());
		System.out.println("Boot classloader: " + applicationClassLoader.getParent().getParent());
	}
}
Current classloader: sun.misc.Launcher$AppClassLoader@73d16e93
Parent classloader: sun.misc.Launcher$ExtClassLoader@15db9742
Boot classloader: null //启动加载器是通过C++实现的，用于加载${JAVA_HOME}/lib中的类。
```

* ClassLoader类分析
1. java.lang.ClassLoader.loadClass(String) 通过类的全限定名，获取Class对象。
2. java.lang.ClassLoader.defineClass(String, byte[], int, int) 通过类的字节码解析并转换成类对象。
3. java.lang.ClassLoader.findSystemClass(String) 从本地的文件系统中获取Class文件并生成Class对象。
4. java.lang.ClassLoader.getParent() 利用双亲委派机制，获取父类加载器。

### Resource对象
> Resource对象是对资源文件的一种抽象，提供了多种对于文件信息获取的方法，可以用于Spring定位包含BeanDefinition的文件并获取资源。

* 两种通过输入流操作获取Resource内容的方法。
```Java
public class ResourceTest {
	public static void main(String[] args) throws IOException {
		InputStream is = null;
		try{
			Resource resource = new FileSystemResource("F://JavaEE_Project/JavaCore/src/beans.xml");
			System.out.println(resource.getFilename());
			System.out.println(resource.getDescription());
			is = resource.getInputStream();
			byte[] b = new byte[1024];
			int index = 0;
			StringBuilder sb = new StringBuilder();
			while((index = is.read(b)) != -1){	//通过输入流的read方法一致读到EOF
				String s = new String(b);
				sb.append(s);
				index = 0;
			}
			System.out.println(sb.toString());
			System.out.println("=====================");
			EncodedResource encRes = new EncodedResource(resource, "UTF-8");
			String content = FileCopyUtils.copyToString(encRes.getReader());	//通过Spring封装好的FileCopyUtils工具类，实现从Reader中获取字符并转换成字符串。
			System.out.println(content);
		}finally{
			is.close();
		}
	}
}
```

### BeanFactory and Application Context
* BeanFactory是Spring的框架设施，就是我们平时所说的IOC容器，是一种较为底层的容器，一般程序员并不会用到，却是Spring实现的核心。
* ApplicationContext叫做应用上下文，面向Spring框架的开发者，有时我们也成为Spring容器。
* 编程时一般都不会用到BeanFactory。

#### BeanFactory
> BeanFactory是类的通用工厂，可以创建并管理各种POJO类，这些类要遵从一定的规范，这样Beanfaactory才能调用反射机制生成类对象。
*  The root interface for accessing a Spring bean container.
```Java
public interface BeanFactory {
	String FACTORY_BEAN_PREFIX = "&";
	Object getBean(String name) throws BeansException;
	<T> T getBean(String name, @Nullable Class<T> requiredType) throws BeansException;
	Object getBean(String name, Object... args) throws BeansException;
	<T> T getBean(Class<T> requiredType) throws BeansException;
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
	boolean containsBean(String name);	//当前容器是否含有包含某种名字的bean
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;	//当前指定的类是否是单例模式，可以在BeanDefinition中指定。
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;	//当前指定的对象是否是原型模式，可以在BeanDefinition中定义。
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;	//确定当前指定的bean是否是指定的Class类。
	boolean isTypeMatch(String name, @Nullable Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;	//获得指定对象属于哪个类。
	String[] getAliases(String name);	//获得指定对象的别名。
}
```

#### 编程式的使用IoC容器
```Java
public class DefaultListableBeanFactoryTest {
	public static void main(String[] args) {
		ClassPathResource resource = new ClassPathResource("beans.xml");	//创建了IoC配置文件的抽象资源，这个资源中包含了BeanDefinition。
		DefaultListableBeanFactory factory = new DefaultListableBeanFactory();	//创建一个beanFactory，实际上是一个spring对象的工厂类，用于生成注册的bean对象
		XmlBeanDefinitionReader xmlReader = new XmlBeanDefinitionReader(factory);	//创建一个BeanDefinition的reader对象。
		xmlReader.loadBeanDefinitions(resource);	//通过reader从xml中加载到beanDefinition。
		Customer customer = (Customer) factory.getBean("customer");
		System.out.println(customer.getName());
	}
}
```

#### ApplicationContext
> ApplicationContext是BeanFactory的实现，并添加了诸多新功能。

1. ApplicationEventPublisher:使容器拥有发布上下文的功能，包括启动事件和关闭事件。
	* ApplicationEventMulticaster的multicastEvent方法可以将事件注册到对应的Listener上。
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
```

2. MessageSource: 用于i18n。
[Spring之国际化实现](https://www.jianshu.com/p/07a3b8e3658a)
	* 需要自己定义MessageSource和Bundle用于填充国际化。

3. ResourcePatternResolver: 用于Resource的匹配.
```Java
public interface ResourcePatternResolver extends ResourceLoader {
	String CLASSPATH_ALL_URL_PREFIX = "classpath*:";	//前缀的正则匹配
	Resource[] getResources(String locationPattern) throws IOException;	//获取多个资源。
}
```
4. ConfigurableApplicationContext：对于ApplicationContext的增强。
5. ApplicationContext会在初始化时将所有的单例全部生成完成，所以会比BeanFactory花更多的时间，但是在后期则会直接调用这个单例，不会再有延时。

#### 使用WebApplicationContext
> 在WebApplicationContext中我们要整合ServletContext和SpringContext。

```Java
/**
	 * Return the standard Servlet API ServletContext for this application.
	 * <p>Also available for a Portlet application, in addition to the PortletContext.
	 */
	ServletContext getServletContext();
```

> 我们必须要等到ServletContext的环境初始化完成后，所以我们需要在web.xml中通过监听器确定ServletContext的环境已经初始化完成。

```xml
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

#### Spring Bean的生命周期
![Spring Bean的生命周期Part1](https://i.imgur.com/4UeZnAx.png)
![Spring Bean的生命周期Part2](https://i.imgur.com/dYhQO2v.png)

* 通过实现InstantiationAwareBeanPostProcessor定义所有的方法。
```Java
public class MyInstantiationAwareBeanPostProcessor implements
		InstantiationAwareBeanPostProcessor {
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName)
			throws BeansException {
		System.out.println("[BeanPostProcessor]:Run postProcessBeforeInitialization");
		return bean;
	}
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName)
			throws BeansException {
		System.out.println("[BeanPostProcessor]:Run postProcessAfterInitialization");
		return bean;
	}
	@Override
	public Object postProcessBeforeInstantiation(Class<?> beanClass,
			String beanName) throws BeansException {
		System.out.println("[InstantiationAwareBeanPostProcessor]:Run postProcessBeforeInstantiation");
		return null;
	}
	@Override
	public boolean postProcessAfterInstantiation(Object bean, String beanName)
			throws BeansException {
		System.out.println("[InstantiationAwareBeanPostProcessor]:Run postProcessAfterInstantiation");
		return true;
	}
	@Override
	public PropertyValues postProcessPropertyValues(PropertyValues pvs,
			PropertyDescriptor[] pds, Object bean, String beanName)
			throws BeansException {
		System.out.println("[InstantiationAwareBeanPostProcessor]:Run postProcessPropertyValues");
		return pvs;
	}
}
```

* 定义要注册的Bean,在其中实现了bean生命周期中的多种方法。
```Java
public class MyBean implements BeanNameAware, BeanFactoryAware, InitializingBean, DisposableBean{
    private BeanFactory beanFactory;
    private String name;
    private String tag;
    public MyBean(){
        System.out.println("调用构造器实例化Bean");
    }
    public void printName(){
        System.out.println("MyBean name is: " + name);
    }
    public String getTag() {
        return tag;
    }
    public void setTag(String tag) {
        this.tag = tag;
        System.out.println("设置Bean的属性为：" + tag);
    }
    @Override
    public void destroy() throws Exception {
        System.out.println("调用DisposableBean接口的destroy方法");
    }
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("调用InitializingBean接口的afterPropertiesSet方法");
    }
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
        System.out.println("调用BeanFactoryAware接口的setBeanFactory方法");
    }
    @Override
    public void setBeanName(String name) {
        this.name = name;
        System.out.println("调用BeanNameAware接口的setBeanName方法");
    }
}
```

* 测试类
```Java
public class LifeCycleTest {
	public static void main(String[] args) {
		Resource resource = new ClassPathResource("beans.xml");
		DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
		beanFactory.addBeanPostProcessor(new MyInstantiationAwareBeanPostProcessor());
		XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
		reader.loadBeanDefinitions(resource);
		MyBean bean = (MyBean) beanFactory.getBean("myBean");
		bean.printName();
		beanFactory.destroySingletons();
	}
}
```

#### 生命周期分析
1. 在调用getBean时，会调用InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation方法。该方法是optional的，需要注册该方法。
	* [InstantiationAwareBeanPostProcessor]:Run postProcessBeforeInstantiation
2. 调用构造器实例化Bean。该构造器方法必须是无参的。
3. 在初始化后，会调用InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation方法，此时对象的实例已经被创建，但是fields都没有被填充。
	* [InstantiationAwareBeanPostProcessor]:Run postProcessAfterInstantiation
4. 在填充属性之前，会调用InstantiationAwareBeanPostProcessor的postProcessPropertyValues。
	* [InstantiationAwareBeanPostProcessor]:Run postProcessPropertyValues
	* 设置Bean的属性为：test
5. 调用BeanNameAware接口的setBeanName方法
	* 此时会将id赋值给类。
6. 调用BeanFactoryAware接口的setBeanFactory方法
	* 将BeanFactory注入给bean对象，此时将IOC容器传入bean。
7. 容器再次获得对Bean进行加工的机会。
	* [BeanPostProcessor]:Run postProcessBeforeInitialization
8. 调用InitializingBean接口的afterPropertiesSet方法，可以再次释放资源，记录日志。
9. 运行BeanPostProcessor的postProcessAfterInitialization方法
	* [BeanPostProcessor]:Run postProcessAfterInitialization
10. 此处开始，bean完成了所有的初始化工作，开始进行对bean的使用。
	* MyBean name is: myBean
11. 调用DisposableBean接口的destroy方法

#### Spring Bean生命周期的讨论
* 实际上上述的绝大多数过程都是可选的，而我们必须通过实现Spring的一些框架。
* 我们希望去除bean和Spring的耦合，所以最好只是让Spring维护POJO。

### IOC容器装配Bean
![Spring 容器高层视图](https://i.imgur.com/SCo3jD2.png)

```XML
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-3.0.xsd">
	<!-- 默认命名空间 -->
 	<bean id="customer" class="ca.mcmaster.spring.Customer" scope="singleton">
		<property name="name" value="Sean"/>
	</bean>
	<bean id="myBean" class="ca.mcmaster.spring.beanfactory.lifecycle.MyBean" scope="singleton">
      <property name="tag" value="test"/>
  </bean>
</beans>
```

#### Dependency Injection依赖注入
1. 属性注入，通过get和set方法将依赖注入到对象中。此处的get/set方法要符合命名规范。
```Java
public class Student {
	private String name;
	private int age;
	private Customer customer;
	public Customer getCustomer() {
		return customer;
	}
	public void setCustomer(Customer customer) {
		this.customer = customer;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public static void main(String[] args) {
		ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml") ;
		Student student = (Student) ctx.getBean("student");
		System.out.println(student.getCustomer().getName());
	}
}
```
```xml
<bean id="student" class="ca.mcmaster.spring.di.Student" scope="singleton">
		<!-- 基本数据类型通过value选项注入 -->
  		<property name="name" value="Sean"/>
  		<property name="age" value="25"/>
		<!-- 引用类型通过ref选项注入 -->
  		<property name="customer" ref="customer"/>
  	</bean>
```

2. 构造器注入
```Java
public class Car {
	private String brand;
	private String color;
	public Car(String brand, String color){	//通过构造器注入
		this.brand = brand;
		this.color = color;
	}
	public String getBrand() {
		return brand;
	}
	public String getColor() {
		return color;
	}
	public void setBrand(String brand) {
		this.brand = brand;
	}
	public void setColor(String color) {
		this.color = color;
	}
	public static void main(String[] args) {
		ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml") ;
		Car car = (Car) ctx.getBean("car");
		System.out.println(car.getBrand());
	}
}
```
```xml
<bean id="car" class="ca.mcmaster.spring.di.Car" scope="singleton">
  	<constructor-arg name="brand" value="RAV4" type="java.lang.String"/>
  	<constructor-arg name="color" value="GREY" type="java.lang.String"/>
</bean>
```
* 在JDK1.8中无法读取Spring3.x的构造器注入法，此处通过JDK1.7可以通过。

3. 工厂方法注入
* 定义工厂类
```Java
public class CarFactory {
	public Car getInstance(){
		Car car = new Car("bmw", "white");
		return car;
	}
	public static void main(String[] args) {
		ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
		Car car = (Car) ctx.getBean("bmw");
		System.out.println(car.getBrand());
	}
}
```
```xml
<!-- 如果是非静态工厂，则需要定义一个工厂类的实例 -->
<bean id="carFactory" class="ca.mcmaster.spring.di.CarFactory" scope="singleton"/>
<!-- 通过工厂类的方法返回实例 -->
 <bean id="bmw" factory-bean="carFactory" factory-method="getInstance"></bean>
```

4. 联级注入
> Spring允许联级注入，但是在内部的引用对象时必须要已经实例化。

* Java属性定义
`private Customer customer = new Customer();`
* xml配置
`<property name="customer.name" value="BBBB"/>`

5. 集合类型属性注入
* List
```java
private List<String> hobbies = new ArrayList<>();	//此处我们要将集合实例化，不然我们要重新定义一个bean，将这个bean传入。
```
```xml
<property name="hobbies">
 	<list>
 		<value>basketball</value>
 		<value>swimming</value>
 		<value>football</value>
 	</list>
 </property>
```
* Set
```Java
private Set<String> set = new HashSet<>();
```
```xml
<property name="set">
  <set>
  	<value>basketball1</value>
  	<value>swimming1</value>
  	<value>football1</value>
  </set>
</property>
```
* Map
```Java
private Map<String, String> map = new HashMap<String, String>();
```
```xml
<property name="map">
	<map>
		<entry key="k1" value="v1"/>
  		<entry key="k2" value="v2"/>
  		<entry key="k3" value="v3"/>
  		<entry key="k4" value="v4"/>
  	</map>
</property>
```
* Properties
```Java
private Properties properties = new Properties();
```
```xml
<property name="properties">
	<props>
		<prop key="name">SeanForFun</prop>
		<prop key="place">McMaster</prop>
	</props>
</property>
```
* 通过spring的utils命名空间添加集合对象
![utils命名空间](https://i.imgur.com/57JRhhF.png)

* 通过look-up method方法注入
> 用于为接口的方法提供默认的返回值，底层是通过CGLib实现的。

```Java
public interface LookupDi {
	public Student getStudent();
}
```
```xml
<bean id="lookupDi" class="ca.mcmaster.spring.di.LookupDi">
	<lookup-method name="getStudent" bean="student"/>
</bean>
```

#### 通过注解实例化和注入
1. 在类上通过`@Component`进行声明。同时还有`@Service`, `@Repository`，`@Configure`等等，这样的类将会被扫描。
2. 在生成引用对象的方法上可以标注`@Bean`，这样会将Bean对象注册到IoC容器中，我们可以通过规则获取对象。

```Java
@Component(value="computer")	//将当前类的对象定义为id为computer的对象，可以定义scope
@Scope("singleton")
public class Computer {
	@Value(value="MacBook")	//通过该注解注入基本数据类型
	private String brand;
	@Autowired	//通过该属性注入对象
	@Qualifier("student")
	private Student student;
	@Resource(name="colorList")
	private List<String> colorList;
	@Bean(name="colorList")	//会将方法的返回值作为一个Bean对象注册到IoC容器中。
	public List<Student> testList(Student student){	//此处student会按照规则注入，此处注入的即为上面的student对象。
		List<Student> testList = new ArrayList<>();
		testList.add(student);
		return testList;
	}
	public List<String> getColorList() {
		return colorList;
	}
	public void setColorList(List<String> colorList) {
		this.colorList = colorList;
	}
	public Student getStudent() {
		return student;
	}
	public void setStudent(Student student) {
		this.student = student;
	}
	public String getBrand() {
		return brand;
	}
	public void setBrand(String brand) {
		this.brand = brand;
	}
	public void printNow(){
		System.out.println("Haha");
	}
	public void printStudent(){
		System.out.println(this.student);
	}
}
```

#### bean的继承
1. 可以通过配置抽象Spring实例并设置共有的属性值。
2. 可以通过配置子对象实例继承自父类。

```xml
<bean id="abstracatCar" class="ca.mcmaster.spring.di.Car" abstract="true">
	<property name="brand" value="RAV4"/>
</bean>
<bean id="redCar" class="ca.mcmaster.spring.di.Car" parent="abstracatCar">
	<property name="color" value="red"/>
</bean>
```

#### bean的依赖
![bean的依赖](https://i.imgur.com/CKRn3L0.png)

#### 多个配置文件之间的引用
![多个配置文件之间的引用](https://i.imgur.com/z19mfSe.png)

#### FactoryBean
> Bean实现FactoryBean，创建对象时，会通过反射机制调用工厂类，并调用getObject方法获得对象。
```Java
public class CarFactoryBean implements FactoryBean<Car> {
	private String carInfo;
	public String getCarInfo() {
		return carInfo;
	}
	public void setCarInfo(String carInfo) {
		this.carInfo = carInfo;
	}
	@Override
	public Car getObject() throws Exception {
		Car car = new Car();
		car.setBrand("TESLA");
		car.setColor("white");
		return car;
	}
	@Override
	public Class<?> getObjectType() {
		return Car.class;
	}
	@Override
	public boolean isSingleton() {
		return false;
	}
}
```

```xml
<bean id="teslaCar" class="ca.mcmaster.spring.di.CarFactoryBean">
	<property name="carInfo" value="This is a while tesla."/>
</bean>
```

#### get/set方法传参
```Java
	@Autowired
	public void setStudent(@Qualifier("student") Student student) {	//会根据配置传入student实例。
		this.student = student;
	}
```

#### @PostConstruct | @PreDestroy
> 在方法内部对生命周期进行影响，会通过注解反射调用所标注的方法。
```Java
@PostConstruct
public void postConstruct(){
	System.out.println("PostConstruct");
}
@PreDestroy
public void preDestroy(){
	System.out.println("preDestroy");
}
```

### IoC容器的初始化
1. BeanDefinition的Resource定位。
* 通过ClassPathResource载入
```Java
ClassPathResource resource = new ClassPathResource("beans.xml");
```

* 通过FileSystemXmlApplicationContext载入
```Java
public class FileSystemXmlApplicationContext extends AbstractXmlApplicationContext {
	public FileSystemXmlApplicationContext() {
	}
	public FileSystemXmlApplicationContext(ApplicationContext parent) {
		super(parent);
	}
	//单个包含BeanDefinition的文件。
	public FileSystemXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}
	//多个包含BeanDefinition的文件。
	public FileSystemXmlApplicationContext(String... configLocations) throws BeansException {
		this(configLocations, true, null);
	}
	public FileSystemXmlApplicationContext(String[] configLocations, ApplicationContext parent) throws BeansException {
		this(configLocations, true, parent);
	}
	public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh) throws BeansException {
		this(configLocations, refresh, null);
	}
	public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
			throws BeansException {
		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();	//启动了BeanDefinition的载入。
		}
	}
	@Override
	protected Resource getResourceByPath(String path) {
		if (path != null && path.startsWith("/")) {
			path = path.substring(1);
		}
		return new FileSystemResource(path);
	}
}
```

* 对IoC容器的初始化
```Java
	protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {	//如果已经存在了一个容器，则将原来的容器关闭。
			destroyBeans();
			closeBeanFactory();
		}
		try {
			DefaultListableBeanFactory beanFactory = createBeanFactory();	//实际上此处持有BeanDefinition的容器就是DefaultListableBeanFactory。
			beanFactory.setSerializationId(getId());
			customizeBeanFactory(beanFactory);
			loadBeanDefinitions(beanFactory);	//载入BeanDefinition
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
	//生成一个IoC容器
	protected DefaultListableBeanFactory createBeanFactory() {
		return new DefaultListableBeanFactory(getInternalParentBeanFactory());
	}
```
2. BeanDefinition的载入。
```Java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();	//抽象方法，子类实现的更新容器的方法。
			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);
			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);
				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);
				// Initialize message source for this context.
				initMessageSource();
				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();
				// Initialize other special beans in specific context subclasses.
				onRefresh();
				// Check for listener beans and register them.
				registerListeners();
				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);
				// Last step: publish corresponding event.
				finishRefresh();
			}
			catch (BeansException ex) {
				logger.warn("Exception encountered during context initialization - cancelling refresh attempt", ex);
				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();
				// Reset 'active' flag.
				cancelRefresh(ex);
				// Propagate exception to caller.
				throw ex;
			}
		}
	}
```
3. 向IoC容器注册BeanDefinition。
