---
layout: post
title:  "Annotation 注解"
date:   2018-11-07 19:56:59
author: Botao Xiao
comment: true
categories: JavaCore
description: 在通过Springboot编写SeanforfunBlog的时候，我大量接触Spring4.x的注解，在一开始学习java的时候我只是粗略的学习了注解的开发和使用，这导致我在对比Spring3.x和Spring4.x的过程中出现了大量的不理解，花费了大量的时间。而从servlet2.5到servlet3.0的升级中，完全抛弃了web.xml的配置而是转为注解式开发更坚定了我想要对注解的研究。所谓磨刀不误砍柴工，我决定暂时停下手上所有的事情从JAVA基础上研究Annotation的原理。
---
# Annotation
>在通过Springboot编写SeanforfunBlog的时候，我大量接触Spring4.x的注解，在一开始学习java的时候我只是粗略的学习了注解的开发和使用，这导致我在对比Spring3.x和Spring4.x的过程中出现了大量的不理解，花费了大量的时间。而从servlet2.5到servlet3.0的升级中，完全抛弃了web.xml的配置而是转为注解式开发更坚定了我想要对注解的研究。所谓磨刀不误砍柴工，我决定暂时停下手上所有的事情从JAVA基础上研究Annotation的原理。

### MetaData 元数据
1. 元数据就是用来定义数据的数据。
2. 例如在数据库中，我们使用的字段就是元数据，而我们存储的数据就是普通的数据。
3. Annotations仅仅是元数据，注解是对于源代码的元数据。

### Why need annotations
1. XML vs. Annotation
2. 注解增强了代码和配置的耦合。XML的大量使用过分分离了代码和元数据的耦合，所以程序员在代码的编写中使用注解增强了代码和元数据的耦合。
3. Annotations仅仅是元数据，和业务逻辑无关。

### 注解的注解-元注解(Meta annotation)
1. @Documented –注解是否将包含在JavaDoc中
	* 一个简单的Annotations标记注解，表示是否将注解信息添加在java文档中。

2. @Retention –什么时候使用该注解
	* 定义该注解的生命周期。
	* RetentionPolicy.SOURCE
		* 在编译阶段丢弃。这些注解在编译结束之后就不再有任何意义，所以它们不会写入字节码。@Override, @SuppressWarnings都属于这类注解。
	* RetentionPolicy.CLASS
		* 在类加载的时候丢弃。在字节码文件的处理中有用。注解默认使用这种方式。
	* RetentionPolicy.RUNTIME
		* 始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解的信息。我们自定义的注解通常使用这种方式。
		* 可以通过使用RUNTIME选项，在运行期间通过反射调用接口中提供的方法。

3. @Target –注解的作用域
	* ElementType.TYPE:用于描述接口、类、枚举、注解
	* ElementType.FIELD:用于描述实例变量
	* ElementType.METHOD：用于描述类的方法
	* ElementType.PARAMETER：用于描述方法的参数
	* ElementType.CONSTRUCTOR：用于描述类的构造器
	* ElementType.LOCAL_VARIABLE：用于描述本地变量
	* ElementType.ANNOTATION_TYPE 另一个注释
	* ElementType.PACKAGE 用于记录java文件的package信息

4. @Inherited – 是否允许子类继承该注解

### 注解定义和使用
1. Annotations只支持基本类型、String及枚举类型。注释中所有的属性被定义成方法，并允许提供默认值。

#### 注解的定义
```Java
@Target(value = { ElementType.METHOD })	//作用在方法上
@Documented()	//添加进入JavaDoc
@Retention(value = RetentionPolicy.RUNTIME)	//生命周期：运行时
@Inherited		//该注解可以被继承
public @interface Todo {
	public enum Priority{LOW, MEDIUM, HIGH};	//可以理解为定义了内部枚举类
	public enum Status{STARTED, NOT_STARTED};
	Priority priority() default Priority.MEDIUM;	//定义了方法，可以给出默认的返回值。
	String author() default "Seanforfun";
	Status status() default Status.NOT_STARTED;
}
```

#### 注解的使用
```Java
public class UseAnnotation {
	@Todo(priority = Priority.LOW, author = "Botao Xiao", status = Status.STARTED)
	public void run(){
		System.out.println("I am not finish.");
	}
}
```

* 通过反射利用注解

```Java
public class StructureUsingAnnotation {
	public static void resolveTodo(){
		Class<UseAnnotation> clazz = UseAnnotation.class;
		Method[] methods = clazz.getDeclaredMethods();	//当前注解是加在方法上的，所以通过反射读取方法，再从方法上读取注解的信息。
		for(Method m : methods){
			if(m.getAnnotation(Todo.class) != null){
				System.out.println("Mehod: " + m.getName());
				Todo a = m.getAnnotation(Todo.class);
				System.out.println("Author: " + a.author());
				System.out.println("Status: " + a.status());
				System.out.println("Priority: " + a.priority());
			}
		}
	}
	public static void main(String[] args) {
		StructureUsingAnnotation.resolveTodo();
	}
}
```

* 结果

```Java
Mehod: run
Author: Botao Xiao
Status: STARTED
Priority: LOW
```

### Reference
1. [Java中的注解是如何工作的](http://www.importnew.com/10294.html)