---
layout: post
title:  "Prototype 原型模型"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: DesignPattern
comment: true
description: 某些对象初始化很复杂，我们在初始化过程中需要进行大量设置，占用大量的资源，所以不使用new而是对已经初始化好的对象进行复制会更快些。在原型模式中我们可以利用过一个原型对象来指明我们所要创建对象的类型，然后通过复制这个对象的方法来获得与该对象一模一样的对象实例。这就是原型模式的设计目的。
---
> 某些对象初始化很复杂，我们在初始化过程中需要进行大量设置，占用大量的资源，所以不使用new而是对已经初始化好的对象进行复制会更快些。
> 在原型模式中我们可以利用过一个原型对象来指明我们所要创建对象的类型，然后通过复制这个对象的方法来获得与该对象一模一样的对象实例。这就是原型模式的设计目的。
![Prototype 原型模型](https://i.imgur.com/CmzDJ8z.png)

### 深克隆浅克隆
1. 使用一个已知实例对新创建实例的成员变量逐个赋值，这个方式被称为浅拷贝。
2. 当一个类的拷贝构造方法，不仅要复制对象的所有非引用成员变量值，还要为引用类型的成员变量创建新的实例，并且初始化为形式参数实例值。
3. 总结一句，就是浅拷贝对引用只会复制引用，而深拷贝会在堆中复制一份对象的实例。
![Deep clone and shallow clone](https://i.imgur.com/9XFNVxw.png)
4. 如何实现深拷贝？
	* 首先对象要实现Cloneable接口，并且重写clone()方法
	* 对于clone()方法，要实现如下几点：
		1. 对任何的对象x，都有x.clone() !=x，即克隆对象与原对象不是同一个对象。（因为是两个对象的实例，所以实例的引用的地址是不同的），所以这两个对象不相同。
		2. 对任何的对象x，都有x.clone().getClass()==x.getClass()，即克隆对象与原对象的类型一样。
		3. 如果对象x的equals()方法定义恰当，那么x.clone().equals(x)应该成立。

### Prototype的实现
1. 该方法实现了Cloneable接口，重写了clone()方法。
```Java
public class Resume implements Cloneable {
	private String name;
	private String gender;
	public Resume(String name, String gender) {
		this.name = name;
		this.gender = gender;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getGender() {
		return gender;
	}
	public void setGender(String gender) {
		this.gender = gender;
	}
	/* (non-Javadoc)
	 * @see java.lang.Object#clone()
	 * @Description 重写了Object的clone()方法，利用了父类的clone方法复制出一个全新的对象。
	 */
	@Override
	protected Object clone(){
		Resume resume = null;
		try {
			resume = (Resume) super.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		return resume;
	}
}
```

2. 使用Prototype,在方法中使用类的clone()方法，生成一个新的对象。
```Java
public class ResumeTest {
	public static void main(String[] args) {
		Resume resume1 = new Resume("Sean Xiao", "male");
		Resume resume2 = (Resume) resume1.clone();
		System.out.println(resume1.getName());
		resume2.setName("Botao Xiao");
		System.out.println(resume1.getName());
		System.out.println(resume2.getName());
	}
}
```
