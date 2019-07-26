---
layout: post
title:  "C++ | Dynamic Memory and Smart Pointers"
date:   2019-07-26 10:06
author: Botao Xiao
categories: C++
comment: true
description: 
---

# Dynamic Memory and Smart Pointers

### Dynamic Memory
Dynamic memory in C++ is the memory assigned by programmer which locates on heap. Programmers are responsible for the life cycle of this memory, which means this memory will not be free unless programmer mannually free it.

Creation and free memory are caused by keyword ```new``` and ```delete```.
```objectivec
int* intPtr = new int;  // for single variable.
delete(intPtr);
int* arrPtr = new int[10];  // for Array.
delete[](arrPtr);
```
    
* new and delete must exist in pair.
    
### Smart Pointer.
The idea of smart pointer is like a pointer with GC. We have two pointer class which are unique_ptr and shared_ptr.

![Imgur](https://i.imgur.com/NkwH8Ar.png)

### shared_ptr
Share ptr maintances a reference count for current pointer, once the value drop to 0, gc.
![Imgur](https://i.imgur.com/o3adB5J.png)
```objectivec
// use shared_ptr pointing to a variable
std::shared_ptr<int> p = make_shared<int>(12);
*p = 22;
cout << *p << endl;
// use shared_ptr pointing to a object.
std::shared_ptr<string> ps = make_shared<string>();
cout << ps << endl;
std::shared_ptr<string> psEmpty;
cout << psEmpty << endl;
*ps = "asdfadsf";
cout << *ps << endl;
cout << *ps.get() << endl;
cout << ps.get() << endl;
// array calls delete[] to free so we must pass our own deleter.
// Not good idea tp control array with smart pointer, better use vector.
shared_ptr<int> parr(new int[10]{}, std::default_delete<int[]>());
int* pt = parr.get();
cout << pt[3] << endl;
pt[3] = 5;
cout << pt[3] << endl;
```

#### Unique_ptr
Once the object holding unique pointer gc, gc all recourses for unique pointer. 
![Imgur](https://i.imgur.com/OsBLlx5.png)

```objectivec
template<typename T>
Stack<T>::Stack(long size) {
    Stack::size = size;
    unique_ptr<T[]> temp(new T[size], std::default_delete<T[]>());
    stack = std::move(temp); // or use stack.reset(temp.release);
    cur = 0;
}
```
    
### Reference
1. C++ Primer edition 5, Chapter 12.
