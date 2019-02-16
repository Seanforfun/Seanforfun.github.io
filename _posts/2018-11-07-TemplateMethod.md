---
layout: post
title:  "Template Method 模板方法"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: DesignPattern
comment: true
description: 定义一个算法中的操作框架，而将一些步骤延迟到子类中。使得子类可以不改变算法的结构即可重定义该算法的某些特定步骤。可以理解成接口方法的延伸。
---
> 定义一个算法中的操作框架，而将一些步骤延迟到子类中。使得子类可以不改变算法的结构即可重定义该算法的某些特定步骤。
> 我认为模板方法和策略模式有些相似，一些策略的实现步骤是固定的，我们就可以定义操作的框架，并通过实现每一小的操作实现整个策略。

### Template Method的实现流程
![Template Method的实现流程](https://i.imgur.com/x55JL55.png)

1. 定义一个接口或是抽象类，被称为抽象模板。
	* 基本方法：是由子类实现的方法，并且在模板方法被调用。（一般都加上final关键字，防止被覆写）
	* 可以有一个或几个，一般是一个具体方法，也就是一个框架，实现对基本方法的调用，完成固定的逻辑。（抽象模板中的基本方法尽量设计为protected类型，符合迪米特法则，不需要暴露的属性或方法尽量不要设置为protected类型。实现类若非必要，尽量不要扩大父类中的访问权限）
```Java
public abstract class TemplateClass {
	public abstract void action1();	//基本方法，由子类实现
	public abstract void action2();
	public final void templateAction(){	//一个具体方法，对基本方法进行调用
		action1();
		action2();
	}
}
```

2. 子类继承抽象模板，实现未实现的方法。
```Java
public class ConcreteMethod1 extends TemplateClass {
	@Override
	public void action1() {
		System.out.println("This is action1 from concrete method1...");
	}
	@Override
	public void action2() {
		System.out.println("This is action2 from concrete method1...");
	}
}
public class ConcreteMethod2 extends TemplateClass {
	@Override
	public void action1() {
		System.out.println("This is action1 from concrete method2...");
	}
	@Override
	public void action2() {
		System.out.println("This is action2 from concrete method2...");
	}
	public static void main(String[] args) {
		TemplateClass template = new ConcreteMethod1();
		template.action1();
		template.action2();
		template = new ConcreteMethod2();
		template.action1();
		template.action2();
	}
}
This is action1 from concrete method1...
This is action2 from concrete method1...
This is action1 from concrete method2...
This is action2 from concrete method2...
```

### 通过Lambda表达式实现模板模式
1. 在Java8中添加了很多的函数式接口，我们可以通过这些函数式接口代替我们自己定义的接口。
2. 接口规范了行为，一些函数式接口可以完全减少了接口的重新定义，并且方便我们传递代码块。

### 总结
1. 封装不变部分，扩展可变部分。把认为不变部分的算法封装到父类中实现，而可变部分的则可以通过继承来继续扩展。
2. 行为由父类控制，子类实现。

### Conclusion
1. [模板方法模式](http://www.cnblogs.com/zhanglei93/p/6021086.html)
