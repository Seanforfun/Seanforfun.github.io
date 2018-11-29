---
layout: post
title:  "Java8| Stream"
date:   2018-11-25 12:32
author: Botao Xiao
categories: JavaCore
description: 从支持数据处理操作的源生成的元素序列。
---
# Stream
从支持数据处理操作的源生成的元素序列。
1. 元素序列：流提供了一个接口，可以访问特定元素类型的一组有序值。集合的目的在于以特定的时间/空间复杂度存储和访问元素，流的目的是为了计算。
2. 源：流会使用一个提供数据的源，如集合，数组或输入输出资源。从有序集合生成流时会保存原有的顺序。
数据处理操作：filter，map, reduce, find, match, sort等操作，都是线程安全的。
3. 流水线：很多流操作都会返回一个流，这样多个操作就能连接起来形成流水线。
4. 内部迭代：流的迭代操作是隐式的，并不像集合需要显式的迭代器。

### Stream的操作
1. filter(), 流水线中的过滤操作。接收一个Predicate对象进行filter
```Java
    /**
     *Returns a stream consisting of the elements of this stream that match
     *the given predicate.
     */
    Stream<T> filter(Predicate<? super T> predicate);
```

2. map(), 接收一个Lambda，将元素转换成其他形式或者提取信息。
```Java
    /**
     *Returns a stream consisting of the results of applying the given
     *function to the elements of this stream.
     */
    <R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

3. limit(), 截断流，使其元素不超过给定数量。
```Java
    /**
     *Returns a stream consisting of the elements of this stream, truncated
     *to be no longer than {@code maxSize} in length.
     */
    Stream<T> limit(long maxSize);
```

4. collect(), 将流转换为其他的形式，例如collect(toList())将流转换成一个链表。
```Java
<R, A> R collect(Collector<? super T, A, R> collector);
```

### 使用流
流的使用分成三件事：
1. 一个数据源(如集合)：执行一个查询。
2. 中间操作链：形成一条流水线。
3. 一个终端操作：执行流水线，生成结果。

#### 使用谓词筛选 filter()方法
```Java
/**
 *Returns a stream consisting of the elements of this stream that match
 *the given predicate.
 *@return the new stream
 */
Stream<T> filter(Predicate<? super T> predicate);
Example:
menu.parallelStream()
        .filter(Dish::isVegetarian)
        .forEach(System.out::println);
```
![Imgur](https://i.imgur.com/O9APuMU.png)

#### 筛选各异的元素 distinct()
```Java
/**
   *Returns a stream consisting of the distinct elements (according to
   *{@link Object#equals(Object)}) of this stream.
   *@return the new stream
   */
  Stream<T> distinct();
  Example：
  list.parallelStream()
        .distinct()
        .forEach(System.out::println);
```
![Imgur](https://i.imgur.com/gftq3ce.png)

#### 截断流 limit(),只会返回固定数值个元素。
```Java
/**
  *Returns a stream consisting of the elements of this stream, truncated
  *to be no longer than {@code maxSize} in length.  
  */
 Stream<T> limit(long maxSize);
menu.parallelStream()
                .filter((d) -> d.getCalories() > 300)
                .limit(3)
                .forEach(System.out::println);
```

#### 跳过元素, skip(n), 会跳过当前流水线中前n个元素。一般和limit结合起来使用。
```Java
menu.parallelStream()
        .filter((d) -> d.getCalories() > 300)
        .skip(2)
        .forEach(System.out::println);
```

#### 对流中的每一个元素应用函数map()
```Java
/**
 *Returns a stream consisting of the results of applying the given
 *function to the elements of this stream.
 *@return the new stream
 */
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

#### 流的扁平化flatMap,实际上就是将数组中的每一个对象生成一个流，即将流扁平化。
```Java
    /**
     *Returns a stream consisting of the results of replacing each element of
     *this stream with the contents of a mapped stream produced by applying
     *the provided mapping function to each element.  Each mapped stream is
     *{@link java.util.stream.BaseStream#close() closed} after its contents
     *have been placed into this stream.  (If a mapped stream is {@code null}
     *an empty stream is used, instead.)
     */
    <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
    Example:
    Arrays.stream(arr)  // 将字符串数组转化成流Stream<String>
        .map(w -> w.split(""))  // 将每个字符串分解成字母，每一个字符串都分解成数组Stream<String[]>
        .flatMap(Arrays::stream)    // 将流扁平化，每一个字符作为字符串都形成一条新的流Stream<String>
        .distinct()
        .forEach(System.out::println);
```
![Imgur](https://i.imgur.com/XraEDix.png)

#### 检查谓词是否匹配一个元素anyMatch(), 终端操作。
```Java
/**
 *Returns whether any elements of this stream match the provided
 *predicate.  May not evaluate the predicate on all elements if not
 *necessary for determining the result.  If the stream is empty then
 *{@code false} is returned and the predicate is not evaluated.
 */
boolean anyMatch(Predicate<? super T> predicate);
Exmaple:
if(menu.stream().anyMatch(Dish::isVegetarian))
    System.out.println("Contains vegetarian food.");
```

#### 检查谓词是否满足所有的元素allMatch(), 终端操作。
```Java
 /**
 *Returns whether all elements of this stream match the provided predicate.
 *May not evaluate the predicate on all elements if not necessary for
 *determining the result.  If the stream is empty then {@code true} is
 *returned and the predicate is not evaluated.
 */
boolean allMatch(Predicate<? super T> predicate);
Example:
System.out.println(menu.stream().allMatch(Dish::isVegetarian));
```

#### 完全不满足noneMatch(), 终端操作。
```Java
/**
 *Returns whether no elements of this stream match the provided predicate.
 *May not evaluate the predicate on all elements if not necessary for
 *determining the result.  If the stream is empty then {@code true} is
 *returned and the predicate is not evaluated.
 */
boolean noneMatch(Predicate<? super T> predicate);
Example:
System.out.println(menu.stream().noneMatch(Dish::isVegetarian));
```

#### 查找复合要求的元素
1. findAny(), 返回一个找到的符合要求的元素。不一定是空间上的第一个。如下的返回值是6
```Java
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9);
list.parallelStream()
        .filter(x -> x % 3 == 0)
        .findAny()
        .ifPresent(System.out::println);
```

2. findFirst(),返回第一个符合要求的元素，必定是空间上的第一个。
```Java
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9);
list.parallelStream()
        .filter(x -> x % 3 == 0)
        .findFirst()
        .ifPresent(System.out::println);
```

#### 规约 reduce
1. 规约将流中的所有的元素反复结合起来最后成为一个值。 reduce(int default, BiFunction bf)
```Java
int result = Arrays.stream(arr1)
        .reduce(0, (a, b) -> a + b);
System.out.println(result);
```
![Imgur](https://i.imgur.com/BM3FWgR.png)

2. reduce(BiFunction bf): 因为没有了默认值作为基础，所以有可能得到null，此时我们将返回值设置成OptionalInt避免空指针问题。
```Java
        int[] arr1 = {1,2,3,4,5,6,7,8};
        OptionalInt result = Arrays.stream(arr1)
                .reduce((a, b) -> a + b);
        if(result.isPresent()) System.out.println(result.getAsInt());
```

### 创建流
1. 通过值创建流
```Java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6, 7, 8);
```

2. 通过数组创建流
```Java
int[] arr = {1,2,3,4,5,6,7,8,9};
System.out.println(Arrays.stream(arr).sum());
```

3. 由文件生成流
```Java
try (Stream<String> lines = Files.lines(Paths.get("E:\\LocalProject\\Concurrent\\src\\main\\java\\ca\\mcmaster\\concurrent\\pojo\\MyReader.java"), Charset.defaultCharset())){
            lines.map(String::length)
                    .forEach(System.out::println);
        }
```

4. 创建无限流,主要有两个静态方法iterate和generate。一般和limit结合起来使用。
```Java
Stream.iterate(0, n -> n + 2)
        .limit(20)
        .forEach(System.out::println);
Stream.generate(Math::random)
        .limit(100)
        .forEach(System.out::println);
```

### 使用流收集数据
#### 规约和汇总
1. 获得流中数据的个数
```Java
Long number = menu.parallelStream().collect(Collectors.counting());
System.out.println(number);
Long number1 = menu.parallelStream().count();
System.out.println(number1);
```

2. 查找流中的最大值和最小值
```Java
// Max
Optional<Dish> max = menu.parallelStream().max((d1, d2) -> d1.getCalories() - d2.getCalories());
Optional<Dish> max1 = menu.parallelStream().collect(Collectors.maxBy((d1, d2) -> d1.getCalories() - d2.getCalories()));
if(max.isPresent()) System.out.println(max.get());
if(max1.isPresent()) System.out.println(max1.get());
// Min
Optional<Dish> max = menu.parallelStream().min((d1, d2) -> d1.getCalories() - d2.getCalories());
Optional<Dish> max1 = menu.parallelStream().collect(Collectors.minBy((d1, d2) -> d1.getCalories() - d2.getCalories()));
if(max.isPresent()) System.out.println(max.get());
if(max1.isPresent()) System.out.println(max1.get());
```

3. 汇总，将流中的某个域进行汇总
```Java
// 计算总和
Integer collect = menu.parallelStream().collect(Collectors.summingInt(Dish::getCalories));
System.out.println(collect);
// 计算平均值
Double collect = menu.parallelStream().collect(Collectors.averagingInt(Dish::getCalories));
System.out.println(collect);
```
![Imgur](https://i.imgur.com/jDimSZu.png)

4. 连接字符串, 首先要确保当前的流已经被映射到了一个字符串流上。
```Java
String collect = menu.parallelStream().map(Dish::getName).collect(Collectors.joining());
System.out.println(collect);
```

#### 广义的规约和汇总， 使用collect和reducing
```Java
    /**
     *Returns a {@code Collector} which performs a reduction of its
     *input elements under a specified mapping function and
     *{@code BinaryOperator}. This is a generalization of
     *{@link #reducing(Object, BinaryOperator)} which allows a transformation
     *of the elements before reduction.
     *@param identity the identity value for the reduction (also, the value that is returned when there are no input elements)
     *@param mapper a mapping function to apply to each input value
     *@param op a {@code BinaryOperator<U>} used to reduce the mapped values
    public static <T, U> Collector<T, ?, U> reducing(U identity,
                                Function<? super T, ? extends U> mapper,
                                BinaryOperator<U> op) {
        return new CollectorImpl<>(
                boxSupplier(identity),
                (a, t) -> { a[0] = op.apply(a[0], mapper.apply(t)); },
                (a, b) -> { a[0] = op.apply(a[0], b[0]); return a; },
                a -> a[0], CH_NOID);
    }
    Example:
    Integer sumCalories = menu.parallelStream().collect(Collectors.reducing(0, Dish::getCalories, (n1, n2) -> n1 + n2));
    System.out.println(sumCalories);
```
* 上一个chapter中所有提及的规约方式均可以通过reducing实现，我们可以通过Function<U, R>来实现mapping， 而我们使用BiFunction来实现我们的规约操作。
* 实现的方法更多了，但是可读性变差了。
![Imgur](https://i.imgur.com/DzUpZFH.png)

#### 分组
分组的返回值是一个Map，对于每一个项目，键是我们定义的classifier，而值是符合这个键的对象的列表。
```Java
Map<Type, List<Dish>> map = menu.stream().collect(Collectors.groupingBy(Dish::getType));
System.out.println(map.toString());
```
![Imgur](https://i.imgur.com/u21zx0w.png)

1. 我们自己手动扩充分组函数
```Java
Map<caloriesType, List<Dish>> collect = menu.stream().collect(Collectors.groupingBy(dish -> {
    if (dish.getCalories() >= 700) return caloriesType.HIGH;
    else if (dish.getCalories() <= 400) return caloriesType.LOW;
    else return caloriesType.MEDIUM;
}));
System.out.println(collect.toString());
```

2. 多级分组
```Java
Map<Type, Map<caloriesType, List<Dish>>> collect = menu.stream().collect(Collectors.groupingBy(Dish::getType, Collectors.groupingBy(dish -> {
    if (dish.getCalories() >= 700) return caloriesType.HIGH;
    else if (dish.getCalories() <= 400) return caloriesType.LOW;
    else return caloriesType.MEDIUM;
})));
System.out.println(collect.toString());
```

3. 将收集器的结果转换成另一种类型
```Java
Map<Type, Dish> collect = menu.stream().collect(Collectors.groupingBy(Dish::getType,
        Collectors.collectingAndThen(Collectors.maxBy(Comparator.comparing(Dish::getCalories)),
                Optional::get)));
System.out.println(collect);
```

#### 分区
分区的好处在于保留了分区函数返回true和false的两套流元素。
```Java
Map<Boolean, Map<Type, List<Dish>>> collect = menu.stream().
        collect(Collectors.partitioningBy(Dish::isVegetarian,
                Collectors.groupingBy(Dish::getType)));
```


















