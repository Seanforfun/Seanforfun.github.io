---
layout: post
title:  "Java8| Lambda Expression"
date:   2018-11-22 15:38
author: Botao Xiao
categories: Javacore
description: Lambda表达式是一种简洁的表示可传递的匿名函数的一种方法：它没有名称，但是它有参数列表，函数主体，返回类型还能抛出异常列表。
---
# Lambda Expression
Lambda表达式是一种简洁的表示可传递的匿名函数的一种方法：它没有名称，但是它有参数列表，函数主体，返回类型还能抛出异常列表。
匿名： 没有明确的名称。
函数： 不像方法属于某个特定的类。
传递： Lambda表达式可以作为参数传递给方法或储存在变量中。
简洁：无需像匿名类写很多模板代码。

### Lambda表达式的构成
```Java
private Comparator<Apple> comparator =
            (Apple a1, Apple a2) -> a1.getWeight() - a2.getWeight();
```
1. 括号：参数列表
2. 箭头： 将参数列表和Lambda主体分隔开
3. Lambda主体：具体的函数给出返回值

### Lambda表达式的语法
1. (parameters) -> expression： 默认expression为返回值。
2. (parameters) ->{statements;}: 在花括号之中填充行为。

### Lambda的使用位置
在函数式接口（一个只定义一个抽象方法的接口）上使用抽象方法。Lambda表达式允许直接以内联的形式为函数式接口的抽象方法提供实现，并把整个表达式作为函数式接口的实例。

### Lambda表达式的实现
1. 行为参数化：所谓的行为参数化是一种策略模式的体现。将一个函数式接口作为一种行为作为参数传递给另一个方法。此时p就是一种行为，对于文件的处理是不固定的，根据接口的实现类而定。典型的策略模式。当前例子中体现在参数FileReaderProcessor p。
```Java
public static String processFile(String file, FileReaderProcessor p) throws IOException {
    File f = new File(file);
    try(BufferedReader br = new BufferedReader(new FileReader(f))) {
        return p.process(br);
    }
}
```

2. 使用函数式接口传递行为。@FunctionalInterface,该接口中只有一个抽象方法。
```Java
@FunctionalInterface
public interface FileReaderProcessor {
    String process(BufferedReader br) throws IOException;
}
```

3. 执行行为。此时的行为是不确定的，需要传入策略。
```Java
return p.process(br);
```

4. 传递Lambda。此处的lambda充当接口的实现类。
```Java
public static void main(String[] args) throws IOException {
    System.out.println(processFile("E:\\LocalProject\\Concurrent\\src\\main\\java\\ca\\mcmaster\\concurrent\\pojo\\UserInfo.java",
            (BufferedReader br) -> {
                return br.readLine() + br.readLine() + br.readLine();
            }));
}
```

### Java8中新增的函数式接口
1. Predicate<T>:判断是否满足条件, 用于添加判断条件。
```Java
@FunctionalInterface
public interface Predicate<T> {
  /**
   *Evaluates this predicate on the given argument.
   *@param t the input argument
   *@return {@code true} if the input argument matches the predicate,
   *otherwise {@code false}
   */
  boolean test(T t);
}
```

2. Consumer<T>: 其中有一个accept方法，用于对某个元素进行操作，无返回值。
```Java
public interface Consumer<T> {
  /**
   *Performs this operation on the given argument.
   *@param t the input argument
   */
  void accept(T t);
}
// 对于每个元素而言，操作是隐藏的。
public static <T> void processList(List<T> list, Consumer<T> c){
    for(T t : list)
        c.accept(t);
}
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8);
processList(list, (Integer i) -> System.out.println(i));  //实际的行为是打印每一个元素。
```

3. Function<T, R>: 其中有一个apply方法，可以接受泛型参数并且定义返回值R类型。这种方法是Consumer方法的延伸，从无返回类型变成了又返回类型。
```Java
@FunctionalInterface
public interface Function<T, R> {
    /**
     *Applies this function to the given argument.
     *@param t the function argument
     *@return the function result
     */
    R apply(T t);
}
public class MyFunctionInterface {
    public static <T, R> List<R> process(List<T> list, Function<T, R> f){
        List<R> result = new LinkedList<>();
        for(T t : list)
            result.add(f.apply(t));
        return result;
    }

    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8);
        List<Integer> result = process(list, (Integer i) -> {return i + 1;});
        for(Integer integer : result)
            System.out.println(integer);
    }
}
```

4. Supplier<T>: 提供对象的函数接口
```Java
@FunctionalInterface
public interface Supplier<T> {
    /**
     *Gets a result.
     *@return a result
     */
    T get();
}
public class MyFunctionInterface {
    public static <T> List<T> createList(Supplier<T> s){
        List<T> result = new LinkedList<>();
        for(int i = 0; i < 10; i++){
            result.add(s.get());
        }
        return result;
    }
    public static void main(String[] args) {
        List<String> str = createList(() -> new String("12234"));
        for (String ss : str)
            System.out.println(ss);
    }
}
```

### 方法引用
方法的引用允许我们重复使用现有的方法定义，并向Lambda一样传递它们。

#### 构建方法的引用
1. 指向静态方法的引用。
![Imgur](https://i.imgur.com/jYLtX9T.png)
2. 指向任意类型实例方法的方法引用。你在引用一个对象的方法，而这个对象本身是Lambda的一个参数。
![Imgur](https://i.imgur.com/UNmKJ0b.png)
3. 指向现有对象的实例方法的引用。在Lambda中引用一个已经存在的外部对象中的方法。
![Imgur](https://i.imgur.com/Ioo89z9.png)

#### 构造函数的引用
1. 无参构造器，使用Supplier<T>的方法。
```Java
public class MyFunctionInterface {
    public static <T> List<T> createList(Supplier<T> s){
        List<T> result = new LinkedList<>();
        for(int i = 0; i < 10; i++){
            result.add(s.get());
        }
        return result;
    }
    public static void main(String[] args) {
        List<String> str = createList(String::new); //引用无参构造方法
        for (String ss : str)
            System.out.println(ss);
    }
}
```

2. 引用带有参数的构造器，使用Function<T,R>函数接口
```Java
public static <T, R> R createObject(T t, Function<T, R> f){
    return f.apply(t);
}
public static void main(String[] args) {
    // 调用了String的带有一个参数的构造器
    String s = createObject("Test", String::new);
    System.out.println(s);
}
```

3. 带有多个参数的构造器，使用BiFunction<T, U, R>函数接口。
```Java
public static <T, U, R> R createObject(T t, U u, BiFunction<T, U, R> f){
    return f.apply(t, u);
}
public static void main(String[] args) {
    String s = createObject("Test", String::new);
    Apple apple = createObject(123, "Apple", Apple::new);
    System.out.println(apple.toString());
}
```

### 复合
#### 比较器的复合Comparator
1. 逆序, 使用reversed方法。
```Java
default Comparator<T> reversed() {
    return Collections.reverseOrder(this);
}
```

2. 链式比较器， thenComparing
```Java
default Comparator<T> thenComparing(Comparator<? super T> other) {
    Objects.requireNonNull(other);
    return (Comparator<T> & Serializable) (c1, c2) -> {
        int res = compare(c1, c2);
        return (res != 0) ? res : other.compare(c1, c2);
    };
}
```

#### 谓词复合 Predicate<T>
1. 非 negate()
```Java
Predicate<Apple> notRedApple = redApple.negate();
```

2. 与 and()
```Java
Predicate<Apple> redAndHeavyApple = redApple.and(a -> a.getWeight() > 150);
```

3. 或 or()
```Java
Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(a -> a.getWeight() > 150).or(a -> "green".equals(a.getColor()));
```

#### 函数复合 Function<T, R>中函数使用
1. andThen()
```Java
public static <T, R> R apply(T t, Function<T, R> f){
    return f.apply(t);
}
public static void main(String[] args) {
    Function<Integer, Integer> f = t -> {return t + 1;};
    Function<Integer, Integer> g = t -> {return t * 2;};
    Function<Integer, Integer> h = f.andThen(g);    // f(g(x))
    Integer res = apply(1, h);
    System.out.println(res);
}
```

2. compose()
```Java
    public static <T, R> R apply(T t, Function<T, R> f){
        return f.apply(t);
    }
    public static void main(String[] args) {
        Function<Integer, Integer> f = t -> {return t + 1;};
        Function<Integer, Integer> g = t -> {return t * 2;};
        Function<Integer, Integer> h = f.compose(g);    // g(f(x))
        Integer res = apply(1, h);
        System.out.println(res);
    }
```

### 引用
1. [Java in Action, current link is temporary, please buy the book from legal path](https://github.com/Seanforfun/Books/blob/master/Java/Java%208%E5%AE%9E%E6%88%98.pdf)













