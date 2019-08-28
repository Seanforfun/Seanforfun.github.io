---
layout: post
title:  "C++ | Pointer and Reference with array"
date:   2019-08-20 12:52
author: Botao Xiao
categories: C++
comment: true
description: 
---
### Introduction
The combinations of pointers and arrays are always confuse, here I want to make a small conclusion about this topic.

### Pointer array 指针数组(Correct)
```int* arr[10];```

* Explanation:
    1. 从右向左理解。(Right to left)
    2. ```arr[10]```: 定义了一个数组，其大小为10。
    3. ```int*```: 定义了数组存储的变量类型，均为指针。
    4. 定义了一个size为10的指针数组。

#### Reference array(Wrong)
```int& arr[10];```

* Explanation:
    1. No this kind of structure, people can just use array itself.

#### Pointer to Array 指向数组的指针(Correct)
```
int arr[10];
int (*ptr)[10] = &arr; 
```

* Explanation:
    1. 定义了一个指针，指向一个含有十个元素的int数组。
    2. 从中间向两侧看。
    3. (*ptr): 定义一个指针。
    4. 其指向的元素是一个数组(*ptr[10])。
    5. 数组中的每一个元素都是int型变量。

#### Reference to Array 数组的引用(Correct)
```objectivec
int arr[10];
int (&ref)[10] = arr;
```

* Explanation:
    1. 定义一个引用，引用一个数组。
    2. (&ref): 定义一个引用。
    3. (&ref)[10]: 该引用指向一个含有10个元素的数组。
    4. int (&ref)[10]: 数组中的每个元素都是int型变量。

#### Reference to pointer array(Correct)
```int * (&ref)[10]```

* Explanation:
    1. (&ref): ref是一个引用(首先括号结合)。
    2. (&ref)[10]: ref引用的对象含有十个元素。
    3. int * (&ref)[10]: 数组中每个元素都是一个指针。
    
### Reference
1. [C++ primer: Chapter 3](http://www.charleshouserjr.com/Cplus2.pdf)