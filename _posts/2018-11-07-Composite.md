---
layout: post
title:  "Composite Pattern 组合模式"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: DesignPattern
description: 组合模式定义了如何将容器对象和叶子对象进行递归组合，使得客户在使用的过程中无须进行区分，可以对他们进行一致的处理。
---
# Composite Pattern 组合模式
> 组合模式定义了如何将容器对象和叶子对象进行递归组合，使得客户在使用的过程中无须进行区分，可以对他们进行一致的处理。

### 如何实现
1. 所有的对象都继承同一个接口，而这个接口定义了所有实例都要实现的方法。
2. 定义一个组合类，在组合类中定义一个数据结构用于承载这些实例。
3. 组合实现方法，遍历数据结构，实现每个实例的方法。

### Composition Pattern
1. 定义一个公共的接口或是抽象类，定义了统一的方法。
```Java
public abstract class File {
	private String name;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public abstract void display();
}
```

2. 定义单独的实例，分别实现了私有的方法。
```Java
public class TextFile extends File {
	public TextFile(String name){
		setName(name);
	}
	@Override
	public void display() {
		System.out.println("This is a text file..." + super.getName());
	}
}
public class ImageFile extends File {
	public ImageFile(String name){
		setName(name);
	}
	@Override
	public void display() {
		System.out.println("This is a image file..." + super.getName());
	}
}
public class VedioFile extends File {
	public VedioFile(String name) {
		setName(name);
	}
	@Override
	public void display() {
		System.out.println("This is a vedio file..." + super.getName());
	}
}
```

3. 定义了一个组合类，通过数据结构承载了所有的实例对象，通过公共接口（或是抽象类）调用每个实例，运行方法。
```Java
public class Folder extends File{
	private final List<File> files;	//定义一个数据结构承载实例。
	public Folder() {
		files = new ArrayList<File>();
	}
	public void add(File f){	//定义方法的增删改查
		files.add(f);
	}
	public void remove(File f){
		files.remove(f);
	}
	@Override
	public void display() {	//遍历对象，使用其方法。
		for(File f : files){
			f.display();
		}
	}
	public static void main(String[] args) {
		Folder folder = new Folder();
		TextFile textFile = new TextFile("Test file");
		folder.add(textFile);
		ImageFile imageFile = new ImageFile("Image file");
		folder.add(imageFile);
		VedioFile vedioFile = new VedioFile("Vedio file");
		folder.add(vedioFile);
		folder.display();
	}
}
```