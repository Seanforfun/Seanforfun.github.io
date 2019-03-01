---
layout: post
title:  "Design Pattern | Flyweight Pattern"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: DesignPattern
comment: true
description: 享元模式的主要目的是实现对象的共享，即共享池，当系统中对象多的时候可以减少内存的开销，通常与工厂模式一起使用。采用一个共享来避免大量拥有相同内容对象的开销。这种开销中最常见、直观的就是内存的损耗。享元模式以共享的方式高效的支持大量的细粒度对象。共享的对象必须是不可变的，不然一变则全变（如果有这种需求除外）。
---
# Flyweight Pattern 享元模式
> 我个人认为，享元模式最常用的体现就是池化技术，例如线程池，连接池。
> 享元模式的主要目的是实现对象的共享，即共享池，当系统中对象多的时候可以减少内存的开销，通常与工厂模式一起使用。
> 采用一个共享来避免大量拥有相同内容对象的开销。这种开销中最常见、直观的就是内存的损耗。享元模式以共享的方式高效的支持大量的细粒度对象。
> 共享的对象必须是不可变的，不然一变则全变（如果有这种需求除外）。

### 内部状态和外部状态
1. 内部状态：在享元对象内部不随外界环境改变而改变的共享部分。
2. 外部状态：随着环境的改变而改变，不能够共享的状态就是外部状态。
3. 内部状态存储于享元对象内部，而外部状态则应该由客户端来考虑。

### 享元模式流程
![Flyweight Pattern](https://i.imgur.com/RoDOOQZ.png)

1. 定义一个Flyweight工厂模式，在工厂模式中定义一个数据结构用于存储可以实例化的对象。
2. 在向工厂模式请求生产实例对象时，先在数据结构中获取对象。
3. 如果对象不存在，我们就新建一个实例对象，存入数据结构，如果存在，则直接取出实例。

### 享元模式实现
1. 定义一个接口类。
```Java
public interface Shape {
	public void draw();
}
```

2. 定义可以类，实现接口。
```Java
public class Circle implements Shape {
	private String color;
	public Circle(String color) {
		this.color = color;
	}
	@Override
	public void draw() {
		System.out.println("Draw a "+ this.color + " circle.");
	}
}
```

3. 定义享元模式的工厂类
```Java
public class FlyweightFactory {
	private static Map<String, Shape> shapes;	//定义一个数据结构并实例化，用于存储已经实例化后的对象。
	static{
		shapes = new HashMap<String, Shape>();
	}
	public static Shape getShape(String key){	//获取一个对象
		Shape shape = shapes.get(key);
		if(null == shape){	//如果对象不存在，则实例化一个对象，并将对象存入数据结构中
			shape = new Circle(key);
			shapes.put(key, shape);
		}
		return shape;	//返回新创建的对象或是从数据结构中取出的对象。
	}
	public static int getSum(){
		return shapes.size();
	}
	public static void main(String[] args) {
		Shape grey = FlyweightFactory.getShape("grey");
		grey.draw();
		System.out.println(grey);
		Shape grey1 = FlyweightFactory.getShape("grey");
		System.out.println(grey1);
		Shape red = FlyweightFactory.getShape("red");
		red.draw();
	}
}
Draw a grey circle.
ca.mcmaster.oopdesign.flyweight.Circle@15db9742	//发现工厂中取出的对象是相同的。说明没有新的创建。
ca.mcmaster.oopdesign.flyweight.Circle@15db9742
Draw a red circle.
```
