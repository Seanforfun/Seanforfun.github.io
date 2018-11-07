---
layout: post
title:  "Abstract Factory 抽象工厂"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: DesignPattern
description: 抽象工厂意图修复工厂方法的一个缺点：一个具体工厂只能创建一类产品。提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类；具体的工厂负责实现具体的产品实例。
---
> 抽象工厂意图修复工厂方法的一个缺点：一个具体工厂只能创建一类产品
> 抽象工厂提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类；具体的工厂负责实现具体的产品实例。

### 使用步骤
1. 创建抽象工厂类，定义具体工厂的公共接口；
2. 创建抽象产品族类 ，定义抽象产品的公共接口；
3. 创建抽象产品类 （继承抽象产品族类），定义具体产品的公共接口；
4. 创建具体产品类（继承抽象产品类） & 定义生产的具体产品；
5. 创建具体工厂类（继承抽象工厂类），定义创建对应具体产品实例的方法；
6. 客户端通过实例化具体的工厂类，并调用其创建不同目标产品的方法创建不同具体产品类的实例

### 实现
* 需求：在原有的两家塑料厂里增设生产需求的功能，即A厂能生产容器+模具产品；B厂也能生产模具+容器产品。
1. 创建抽象工厂类，定义具体工厂的公共接口
```Java
//工厂可以生成多种产品
public interface Factory {
	public Product produceContainer();
	public Product produceModule();
}
```

2. 创建抽象产品族类，定义抽象产品的公共接口；
```Java
public interface Product {
	public void show();
}
```

3. 创建抽象产品类 （继承抽象产品族类），定义具体产品的公共接口
	* 定义一种产品，例如模具类或者容器类，但是仍是一种抽象的概念，不能实例化。
```Java
//抽象的容器产品类
public abstract class ContainerProduct implements Product {
	@Override
	public abstract void show();
}
//抽象的模具产品类
public abstract class MouldProduct implements Product {
	@Override
	public abstract void show();
}
```

4. 创建具体产品类（继承抽象产品类） & 定义生产的具体产品
```Java
public class ContainerProductA extends ContainerProduct {
	@Override
	public void show() {
		System.out.println("I am ContainerProductA.");
	}
}
public class ContainerProductB extends ContainerProduct {
	@Override
	public void show() {
		System.out.println("I am ContainerProductB.");
	}
}
public class MouldProductA extends MouldProduct {
	@Override
	public void show() {
		System.out.println("I am MouldProductA");
	}
}
public class MouldProductB extends MouldProduct {
	@Override
	public void show() {
		System.out.println("I am MouldProductB");
	}
}
```

5. 创建具体工厂类（继承抽象工厂类），定义创建对应具体产品实例的方法
```Java
// 工厂A会生成两种类型的产品。
public class FactoryA implements Factory {
	@Override
	public Product produceContainer() {
		return new ContainerProductA();
	}
	@Override
	public Product produceModule() {
		return new MouldProductA();
	}
}
//工厂B会生成两种类型的产品。
public class FactoryB implements Factory {
	@Override
	public Product produceContainer() {
		return new ContainerProductB();
	}
	@Override
	public Product produceModule() {
		return new MouldProductB();
	}
}
```

6. 客户端通过实例化具体的工厂类，并调用其创建不同目标产品的方法创建不同具体产品类的实例
```Java
public class Test {
	public static void main(String[] args) {
		Factory factoryA = new FactoryA();
		factoryA.produceContainer().show();
		factoryA.produceModule().show();
		Factory factoryB = new FactoryB();
		factoryB.produceContainer().show();
		factoryB.produceModule().show();
	}
}
I am ContainerProductA.
I am MouldProductA
I am ContainerProductB.
I am MouldProductB
```

### Reference
1. [抽象工厂模式（Abstract Factory）- 最易懂的设计模式解析](https://blog.csdn.net/carson_ho/article/details/54910287)