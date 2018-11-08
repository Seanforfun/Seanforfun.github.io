---
layout: post
title:  "Spring高级属性"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: JavaCore Concurrent
description: Spring的高级属性讲解。
---
# Spring高级属性

### Spring属性编辑器
1. 属性编辑器是用于将传入的字符串，或数字等等转换成我们需要的属性形式。
2. Spring为我们提供了两种属性转换的方法。
	* 使用自定义属性编辑器通过继承PropertyEditorSupport，重写setAsText方法。
```Java
/**
 * @author SeanForFun E-mail:xiaob6@mcmaster.ca
 * @date Oct 3, 2018 2:40:06 PM
 * @version 1.0
 */
public class CustomDateFormat extends PropertyEditorSupport{
	private String format = "yyyy-MM-dd";
	public void setFormat(String format) {
		this.format = format;
	}
	@Override
	public void setAsText(String text) throws IllegalArgumentException {
		SimpleDateFormat format = new SimpleDateFormat(this.format);
		try {
			this.setValue(format.parse(text));
		} catch (ParseException e) {
			e.printStackTrace();
		}
	}
}
```
```xml
        <!-- 自定义属性编辑器, 将转换器注册到了CustomEditorConfigurer的Map属性中 -->
	<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
		<!-- private Map<Class<?>, Class<? extends PropertyEditor>> customEditors;private Map<Class<?>, Class<? extends PropertyEditor>> customEditors; -->
		<property name="customEditors">
			<map>
				<entry key="java.util.Date">
					<bean class="com.spring.DatePropertyEditor">
			          	<property name="format" value="yyyy-MM-dd"/>
			        </bean>
				</entry>
			</map>
		</property>
	</bean>
```

	* 通过注册Spring自带的属性编辑器CustomDateEditor。
```Java
/**
 * @author SeanForFun E-mail:xiaob6@mcmaster.ca
 * @date Oct 3, 2018 2:52:02 PM
 * @version 1.0
 */
public class DateFormatRegister implements PropertyEditorRegistrar{

	@Override
	public void registerCustomEditors(PropertyEditorRegistry registry) {
		registry.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), false));
	}
}
```

```Xml
 	<!-- 注册Spring自带编辑器, 将属性转换器注册到链表中 -->
 	<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
 		<property name="propertyEditorRegistrars">
 			<list>
 				<bean class="com.bjsxt.spring.DatePropertyEditorRegistrar"></bean>
 			</list>
 		</property>
 	</bean>
```

### Spring使用资源文件
使用资源文件可以将一些常量信息和bean进行隔离，以便于协同工作和整体维护。
1. 引用Properties文件
	* 通过context命名空间引用properties文件，较为简单，推荐
```xml
    <!-- 引入properties文件 -->
    <context:property-placeholder location="classpath:jdbc.properties" file-encoding="utf-8"/>
	<!-- 使用properties文件的内容-->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    	<property name="driverClassName" value="${driverClassName}"/>
    	<property name="url" value="${url}"/>
    	<property name="username" value="${username}"/>
    	<property name="password" value="${password}"/>
    </bean>
```

	* 通过PropertyPlaceholderConfigurer引入文件
```xml
	<!-- 通过PropertyPlaceholderConfigurer引入properties文件 -->
	<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer" scope="singleton">
    	<property name="location" value="classpath:jdbc.properties"/>
    </bean>
```

### 国际化i18n
国际化中最重要的是定义一个Locale对象，在构造Formater对象的时候传入我们所需要的Locale对象，通过format方法就能获得相应的国际化对象。
1. NumberFormat
```Java
public class MyNumberFormat {
	public static void main(String[] args) {
		Locale ch = new Locale("zh", "CN");
		NumberFormat zhnf = NumberFormat.getCurrencyInstance(ch);
		Locale ca = new Locale("en", "CA");
		NumberFormat canf = NumberFormat.getCurrencyInstance(ca);
		double currency = 12345.6D;
		System.out.println(zhnf.format(currency));
		System.out.println(canf.format(currency));
	}
}
￥12,345.60
$12,345.60
```

2. MessageFormat
```Java
/**
 * @author SeanForFun E-mail:xiaob6@mcmaster.ca
 * @date Jul 19, 2018 9:48:25 AM
 * @version 1.0
 */
public class MyMessageFormat {
	public static void main(String[] args) {
		// 信息格式化串
		String chMsg = "{0}, 你好！你于{1}在工商银行存入{2}元！";
		String usMsg = "At {1, time, short} On {1, date,long}, {0} saved {2, number, currency}";
		// 定义动态占位符的参数
		Object[] params = {"Sean", new GregorianCalendar().getTime(), 1000000000};
		// 指定国际化信息，定义formatter
		MessageFormat zhMf = new MessageFormat(chMsg, Locale.CHINESE);
		//装配参数
		String zhPattern = zhMf.format(params);
		System.out.println(zhPattern);
		MessageFormat usMf = new MessageFormat(usMsg, Locale.CANADA);
		String usPattern = usMf.format(params);
		System.out.println(usPattern);
	}
}
```

3. 使用ResourceBundle来存储可能被复用的国际化信息
	* 命名方式： <资源名>_<语言代码>_<国家地区代码>.properties
	```properties
	greeting.common=Hello!
	greeting.morning=Good morning!
	greeting.night=Good night!
	```

	* 字符串properties文件
	```properties
	greeting.common=Hello {0}! Today is {1}!
	greeting.morning=Good morning {0}! Now is {1, time, short}!
	greeting.night=Good night {0}! Now is {1, date, long}!
	```

	* 在程序中调用tokens和要插入的字符串
	```Java
	public class MyResourceBundle {
	public static void main(String[] args) throws UnsupportedEncodingException {
		ResourceBundle enBundle = ResourceBundle.getBundle("ca/mcmaster/spring/i18n/greeting", Locale.CANADA);
		ResourceBundle cnBundle = ResourceBundle.getBundle("ca/mcmaster/spring/i18n/greeting", Locale.CHINESE);
		System.out.println(enBundle.getString("greeting.common"));
		System.out.println(new String(cnBundle.getString("greeting.common").getBytes("ISO-8859-1"), "utf-8"));
		ResourceBundle fmtBundle = ResourceBundle.getBundle("ca/mcmaster/spring/i18n/fmt_greeting", Locale.CANADA);
		MessageFormat msgFormat = new MessageFormat(fmtBundle.getString("greeting.common"), Locale.CANADA);
		Object[] params = {"Sean", new GregorianCalendar().getTime()};
		String formatMsg = msgFormat.format(params);
		System.out.println(formatMsg);
	}
}
	```

### 容器事件体系

### Reference
1. [Spring中属性编辑器的使用](https://blog.csdn.net/cjj3930337/article/details/8188744)