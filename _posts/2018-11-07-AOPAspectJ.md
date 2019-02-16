---
layout: post
title:  "AspectJ in Spring"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: Spring
comment: true
description: Spring与Aspect整合实现。
---
# AspectJ in Spring

### Annotation in Java
> My previous conclusion about annotation:
> [Annotation V2.0](https://github.com/Seanforfun/JavaCore/blob/master/Conclusions/Annotation.md)

#### 注解中域的数据类型
1. 字符串
2. 布尔型
3. 枚举
4. 注解
5. 上述各种类型的数组形式

* 定义一个注解
```Java
public @interface Review {
	Grade value();
	String reviewer();
}
```

* 注解域以及其反射调用
```Java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Inherited
public @interface Reviews {
	Review[] value();	//虽然被叫做域可仍然是一个方法，在反射中通过方法调用。
}
```

* 反射中调用注解并读取信息
```Java
@Reviews({@Review(value=Grade.EXCELLENT, reviewer="Seanforfun"),
		@Review(value=Grade.UNSATISFACTORY, reviewer="Sean")})
	public static void annotationFieldTest() throws Exception{
		Method testMethod = AnnotationTest.class.getDeclaredMethod("annotationFieldTest", null);
		Annotation[] annotations = testMethod.getDeclaredAnnotations();
		for(Annotation annotation : annotations){
			Method method = annotation.getClass().getDeclaredMethod("value", null);
			Review[] reviews = (Review[]) method.invoke(annotation, null);
			for(Review r : reviews){
				System.out.println(r);
			}
		}
	}
在main函数中调用获得的结果：
@ca.mcmaster.spring.aop.annotation.Review(value=EXCELLENT, reviewer=Seanforfun)
@ca.mcmaster.spring.aop.annotation.Review(value=UNSATISFACTORY, reviewer=Sean)
```

### AspectJ
#### AspectJ定义切面
```Java
@Aspect		//	Define this class as an aspect
public class PreGreetingAspect {
	// 定义了切点
	@Before("execution(* ca.mcmaster.spring.aop..*.greetTo(..))")
	public void beforeGreeting(){	//定义了增强的内容
		System.out.println("How do you do?");
	}
}
@Test
	public void test() throws Exception {
		NaiveWaiter waiter = new NaiveWaiter();
		AspectJProxyFactory factory = new AspectJProxyFactory();	//AspectJ的代理工厂。
		factory.setTarget(waiter);	//设置要代理的对象
		factory.addAspect(PreGreetingAspect.class);	//向对象中织入切面，通过反射机制获得切点和增强。
		Waiter proxy = factory.getProxy();	//获得一个被增强的对象
		proxy.greetTo("Sean");
		proxy.serve("Sean");
	}
How do you do?
Greet to Sean
```

#### 通过xml配置使用AspectJ
```xml
<!-- 配置AspectJ -->
<!-- 配置AspectJ切面bean -->
<bean class="ca.mcmaster.spring.aop.wiring.PreGreetingAspect"/>
<!-- 该bean会自动扫描@AspectJ注解, 同时也会自动织入所有通过xml配置的切面 -->
<bean class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator"/>
```

#### 通过schema进行自动代理装配
```xml
<!--只会扫描属性上的注解-->
<context:annotation-config></context:annotation-config>
```

#### AspectJ作用于方法的表达式
1. execution(): 表达式主体。
2. 第一个*号：表示返回类型，*号表示所有的类型。
3. 包名：表示需要拦截的包名，后面的两个句点表示当前包和当前包的所有子包，com.sample.service.impl包、子孙包下所有类的方法。
4. 第二个*号：表示类名，*号表示所有的类。
5. *(..):最后这个星号表示方法名，*号表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数。

#### 不同的增强类型
* Before
![Before](https://i.imgur.com/oWhTwja.png)
![Before](https://i.imgur.com/qERbrtX.png)
* AfterReturning
![AfterReturning](https://i.imgur.com/OxwEcwu.png)
* Around
![Around](https://i.imgur.com/8Dwe515.png)
* AfterThrowing
![AfterThrowing](https://i.imgur.com/3QUNGiZ.png)
* After
![After](https://i.imgur.com/ykUiRi8.png)
* DeclareParents
![DeclareParents](https://i.imgur.com/6Z6rBEA.png)

#### @annotation
* @annotation注解起到了和execution类似的作用，aspectJ命名空间会自动扫描包含该注解的方法并进行方法的织入。
```Java
@AfterReturning("@annotation(ca.mcmaster.spring.aop.annotation.NeedTest)")
	public void needTestFun(){
		System.out.println("needTestFun() called");
	}
```

#### 切点的复合运算
1. 与运算&&， 可以通过定义两个execution表达书并通过&&连接，则能只对两个表达式的交集进行运算。
2. 或表达式||，可以对两个execution表达式的并集进行运算。
3. 非表达式！， 非表达式大多用于&&或||表达式中某个运算单元取反，很少单独使用。

#### 织入的顺序
1. 在同一个切面中，按照增强在切面中定义的顺序进行织入。
2. 不同的切面中，如果均实现了org.springframework.core.Ordered方法，就会按照顺序从小到大执行织入。
3. 如果位于不同的切面中，并且均没有实现org.springframework.core.Ordered方法，则织入的顺序是不确定的。

#### 连接点的信息（JointPoint）
> 我们之所以要使用jointpoint是因为我们想实现环绕式增强。
> 想要实现环绕式增强需要自己实现public void jointPointAccess(ProceedingJoinPoint pjp) throws Throwable方法并在其中调用方法运行。并在方法运行前后分别实现增强。

```Java
@Around("execution(* greetTo(..))")
	public void jointPointAccess(ProceedingJoinPoint pjp) throws Throwable{
		System.out.println("------------Before----------------");	//前置增强的逻辑
		pjp.proceed();	//连接点的执行
		System.out.println("------------After----------------");	//后置增强的逻辑。
	}
------------Before----------------
Greet to Seanforfun
------------After----------------
```

#### 绑定连接点方法入参
> 使用target方法传入参数
> 重载public void bindJoinPointParams(...)并传入参数以便在增强中调用。

![绑定连接点方法入参](https://i.imgur.com/zIsIeFs.png)
```Java
//
@Before("target(ca.mcmaster.spring.aop.NaiveWaiter) && args(num, name)")//前置增强
public void bindJoinPointParams(int num, String name){	//说明传入的参数是num，name, 是通过参数名进行匹配的。
	System.out.println("Get name" + name);
	System.out.println("Get num" + num);
}
```

#### 绑定代理对象
> 通过this实现绑定，会自动检测类型会生成代理。进行织入。

![](https://i.imgur.com/J9Uf9BQ.png)
```Java
@Before("this(waiter)")
public void bindProxyObj(Waiter waiter){
	System.out.println("----------------------bindProxyObj()-------------------");
	System.out.println(waiter.getClass().getName());
}
----------------------bindProxyObj()-------------------
com.sun.proxy.$Proxy19	//实现接口的对象，所以会通过java proxy实现。
```

#### 绑定类注解对象
