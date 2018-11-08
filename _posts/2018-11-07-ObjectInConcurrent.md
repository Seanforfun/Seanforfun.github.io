---
layout: post
title:  "Publish Object 发布对象"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: Concurrent JVM
comment: true
description: 对象不正确的发布会造成线程的不安全，所以本文总结了对象不安全发布的例子和几种安全发布对象的方法。
---
对象不正确的发布会造成线程的不安全，所以本文总结了对象不安全发布的例子和几种安全发布对象的方法。

### 不安全的发布对象
```Java
@ThreadNotSafe
public class PublishObject {
    @Getter
    private Integer[] arr = {1,2,3,4};

    public static void main(String[] args) {
        Integer[] arr = new PublishObject().getArr();
        System.out.println(Arrays.toString(arr));
        arr[2] = 8;
        System.out.println(Arrays.toString(arr));
    }
}
```
此处的arr数组虽然是私有的，但是有getter方法可以从对象以外的地方获取，所以无法保证此时的arr对象是否已经被当前线程外的其他线程所修改，造成线程不安全。这就是最典型的不安全发布对象。


#### 使用懒汉模式发布
* 不安全的懒汉模式
```Java
@ThreadNotSafe
public class SingletonExample1 {
    /**
     * 使用私有构造器保证外部无法调用构造器实现单例模式。
     */
    private  SingletonExample1(){}

    /**
     * 使用静态变量持有单例。
     */
    private static SingletonExample1 instance = null;

    /**
     * JVM创建一个新的对象实例的步骤：
     * 1. memory = allocate() ; 在Java堆中开辟一块空间。
     * 2. ctorInstance(); //初始化对象，赋值等等操作。
     * 3. instance = memory； 将实例的引用指向该块内存区。
     *
     * 其中步骤2和步骤3之间是没有先后顺序的，所以JVM可能会指令重排。
     * 1. memory = allocate() ; 在Java堆中开辟一块空间。
     * 3. instance = memory； 将实例的引用指向该块内存区。
     * 2. ctorInstance(); //初始化对象，赋值等等操作。
     *
     * 一个线程因为指令重排正在执行第三步而第二步还没有进行，
     * 另一个线程已经可以拿到当前引用并返回该引用。此时该引用还没有
     * 初始化完毕，就会造成问题。
     * @return
     */
    @ThreadNotSafe("如果instance对象没有被volatile修饰则不是线程安全")
    @Discription("双重检测->懒汉单例模式")
    public static SingletonExample1 getInstance(){
        if(instance == null){
            synchronized (SingletonExample1.class){
                if(instance == null)
                    instance = new SingletonExample1();
            }
        }
        return instance;
    }
}
```

* 安全的懒汉模式
```Java
@ThreadSafe
public class SingletonExample1 {
    /**
     * 使用私有构造器保证外部无法调用构造器实现单例模式。
     */
    private  SingletonExample1(){

    }

    /**
     * 使用静态变量持有单例。
     * volatile + 双重检测机制 = 禁止指令重排
     */
    private volatile static SingletonExample1 instance = null;

    /**
     * JVM创建一个新的对象实例的步骤：
     * 1. memory = allocate() ; 在Java堆中开辟一块空间。
     * 2. ctorInstance(); //初始化对象，赋值等等操作。
     * 3. instance = memory； 将实例的引用指向该块内存区。
     * volatile变量保证了JVM不会进行指令重排。
     * @return
     */
    @ThreadSafe
    @Discription("双重检测->懒汉单例模式")
    public static SingletonExample1 getInstance(){
        if(instance == null){
            synchronized (SingletonExample1.class){
                if(instance == null)
                /**
                 * volatile保证了指令重排，确定了线程安全。
                 */
                    instance = new SingletonExample1();
            }
        }
        return instance;
    }
}
```

#### 使用饿汉模式进行发布
```Java
@ThreadSafe
@Discription("饿汉模式->在类被加载的阶段就会被初始化" +
        "类加载器保证了线程安全，但是会造成效率的下降")
public class SingletonExample2 {
    private SingletonExample2(){};
    private static SingletonExample2 instance = new SingletonExample2();
    public static SingletonExample2 getInstance(){
        return instance;
    }
}
```

#### 使用枚举模式实现单例发布
```Java
@ThreadSafe
@Discription("通过枚举模式实现线程安全的单例模式")
public class SingletonExample3 {
    private SingletonExample3(){};
    public static SingletonExample3 getInstance(){
        return Singleton.INSTANCE.getInstance();
    }
    private enum Singleton{
        INSTANCE;
        private SingletonExample3 singleton;
        /**
         * 枚举的构造器，JVM会保证该构造器只会被调用一次
         */
        Singleton(){
            singleton = new SingletonExample3();
        }
        public SingletonExample3 getInstance(){
            return singleton;
        }
    }
}
```

### 引用
1. [5 2 安全发布对象 四种方法 1](https://www.youtube.com/watch?v=Wf2s4HXa9vI&list=PLfezYSPixwxv7Y5FcrWXSQqX9vBF8OLMA&index=44)
