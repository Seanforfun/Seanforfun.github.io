---
layout: post
title:  "Design Pattern | Decorator Pattern"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: DesignPattern
comment: true
description: 装饰模式就是给一个对象增加一些新的功能，而且是动态的，要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例.通过装饰者模式对方法进行增强，通过对实例方法的调用，可以在调用之前或是之后实现增强，实现AOP。
---
# Decorator Pattern 装饰者模式
> 装饰模式就是给一个对象增加一些新的功能，而且是动态的，要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例.
> 通过装饰者模式对方法进行增强，通过对实例方法的调用，可以在调用之前或是之后实现增强，实现AOP。

### Proccess of Decorator
1. 定义一个接口，其中定义了我增强的方法。
```Java
public interface Sourcable {
	public default void method(){
		System.out.println("This is a method...");
	}
}
```

2. 定义了一个装饰器，继承了相同的接口，在接口中接收要增强对象的实例，并重写要增强的方法，通过对实例该方法的调用，在调用之前和之后实现增强。
```Java
public class Decorator implements Sourcable{
	private Sourcable source;	//接收一个要增强对象的实例。
	public Decorator(Sourcable source){
		this.source = source;
	}
	private void adviceBefore(){	//前置增强
		System.out.println("This is method before...");
	}
	private void adviceAfter(){	//后置增强
		System.out.println("This is method after...");
	}
	@Override
	public void method() {	//重写要增强的方法
		adviceBefore();
		source.method();	//对方法本身进行调用
		adviceAfter();
	}
	public static void main(String[] args) {
		Decorator decorator = new Decorator(new Sourcable() {
		});
		decorator.method();
	}
}
This is method before...
This is a method...
This is method after...
```

### 总结
1. 个人认为这是一种AOP的实现，但是是在运行时确定的。
2. 对方法进行了横向的增强。
