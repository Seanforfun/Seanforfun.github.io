---
layout: post
title:  "Factory Method 简单工厂模式"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: DesignPattern
comment: true
description: 介绍了简单工厂模式。
---
# Factory Method

### Simple Factory Method 简单工厂模式
> 简单工厂模式分为三种：普通简单工厂、多方法简单工厂、静态方法简单工厂

#### 普通简单工厂 Normal Simple Factory
> 就是建立一个工厂类，对实现了同一接口的一些类进行实例的创建。

![普通简单工厂](https://i.imgur.com/THK0ggI.png)

1. 定义接口
```Java
public interface Sender {
	public default void send(){
		System.out.println("This is a common sender!");
	}
}
```

2. 实现类实现接口
	* 定义一个MailSender实现类
	```Java
	public class MailSender implements Sender {
		@Override
		public void send() {
			System.out.println("This is a Mail Sender!");
		}
	}
	```

	* 定义一个SMSSender实现类
	```Java
	public class SMSSender implements Sender {
		@Override
		public void send() {
			System.out.println("This is a SMS Sender!");
		}
	}
	```

3. 定义一个工厂方法，用于接收信号，生成不同的Sender类型
```Java
public class SenderFactory {
	public Sender produce(String type){	//接收信号，根据信号的种类生成对象。
		if("mail".equals(type)){
			return new MailSender();
		}else if("sms".equals(type))
			return new SMSSender();
		else
			return new Sender() {
			};
	}
	public static void main(String[] args) {
		SenderFactory senderFactory = new SenderFactory();
		Sender sender = senderFactory.produce(null);
		sender.send();
		Sender mailSender = senderFactory.produce("mail");
		mailSender.send();
		Sender smsMail = senderFactory.produce("sms");
		smsMail.send();
	}
}
```

#### 多方法简单工厂
> 是对普通工厂方法模式的改进，在普通工厂方法模式中，如果传递的字符串出错，则不能正确创建对象，而多个工厂方法模式是提供多个工厂方法，分别创建对象。这种方法减去了传入参数错误的错误。

![多方法简单工厂](https://i.imgur.com/ttOeXYM.png)

1. 创建多方法简单工厂
```Java
public class SimpleMultipleSenderFactory {
	public Sender produceMailSender(){
		return new MailSender();
	}
	public Sender produceSMSSender(){
		return new SMSSender();
	}
	public static void main(String[] args) {
		SimpleMultipleSenderFactory factory = new SimpleMultipleSenderFactory();
		Sender mailSender = factory.produceMailSender();
		mailSender.send();
		Sender smsSender = factory.produceSMSSender();
		smsSender.send();
	}
}
This is a Mail Sender!
This is a SMS Sender!
```

#### 多个静态工厂
> 将上面的多个工厂方法模式里的方法置为静态的，不需要创建实例，直接调用即可。这样就能避免创建工厂实例。

```Java
public class StaticSenderFactory {
	public static Sender produceSMSSender(){
		return new SMSSender();
	}
	public static Sender produceMailSender(){
		return new MailSender();
	}
	public static void main(String[] args) {
		Sender mailSender = StaticSenderFactory.produceMailSender();
		mailSender.send();
		Sender smsSender = StaticSenderFactory.produceSMSSender();
		smsSender.send();
	}
}
```

* 总体来说，工厂模式适合：凡是出现了大量的产品需要创建，并且具有共同的接口时，可以通过工厂方法模式进行创建。在以上的三种模式中，第一种如果传入的字符串有误，不能正确创建对象，第三种相对于第二种，不需要实例化工厂类，所以，大多数情况下，我们会选用第三种——静态工厂方法模式。

### Factory Method 工厂方法
> 简单工厂模式有一个问题就是，类的创建依赖工厂类，也就是说，如果想要拓展程序，必须对工厂类进行修改，这违背了闭包原则，所以，从设计角度考虑，有一定的问题，如何解决？就用到工厂方法模式，创建一个工厂接口和创建多个工厂实现类，这样一旦需要增加新的功能，直接增加新的工厂类就可以了，不需要修改之前的代码。

* 原则是，在生成工厂类的方法也面向接口进行设计。

![工厂方法](https://i.imgur.com/4nEyotp.png)
1. 为工厂定义一个接口，用于生成工厂
```Java
public interface FactoryProvider {
	public Sender produce();
}
```
2. 实现接口，定义出具体的工厂类
```Java
public class SMSSenderFactory implements FactoryProvider {
	@Override
	public Sender produce() {
		return new SMSSender();
	}
}
public class MailSenderFactory implements FactoryProvider {
	@Override
	public Sender produce() {
		return new MailSender();
	}
	public static void main(String[] args) {
		FactoryProvider factory = new SMSSenderFactory();	//面向接口编程，利用多态隐藏了工厂的创建，所以我们可以通过实现接口实现对工厂类的扩展。
		Sender sender = factory.produce();
		sender.send();
	}	//This is a SMS Sender!
}
```

* 其实这个模式的好处就是，如果你现在想增加一个功能：发及时信息，则只需做一个实现类，实现Sender接口，同时做一个工厂类，实现Provider接口，就OK了，无需去改动现成的代码。这样做，拓展性较好！

### Reference
1. [23种设计模式全解析](https://www.cnblogs.com/susanws/p/5510229.html)
