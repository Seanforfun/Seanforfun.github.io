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

* Basic usage of shared_ptr
    ![Imgur](https://i.imgur.com/CBlmsrZ.png)
    ```objectivec
        //make_share会隐式的调用new int(1024)并生成一个指向该地址的shared_ptr.
        shared_ptr<int> ptr = make_shared<int>(1024);
        cout << ptr.unique() << endl; //此处为True.
        // 调用了shared_ptr的explicit构造函数，生成一个新的shared_ptr对象，指向同一块地址
        shared_ptr<int> second(ptr);
        // output: 0x4e7130 0x4e7130,两个指向同一块地址
        cout << ptr << " " << second << endl;
        if(!ptr.unique())
            // 我们给ptr重新分配了一块地址，内容手动做了拷贝。
            ptr.reset(new int(*ptr));
        // output: 0x4e7140 0x4e7130, 指向不同地址
        cout << ptr << " " << second << endl;
        *ptr += 1024;
        // output: 2048 1024
        cout << *ptr << " " << *second << endl;
    ```

* Another Example
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

#### unique_ptr
Once the object holding unique pointer gc, gc all recourses for unique pointer. 
![Imgur](https://i.imgur.com/OsBLlx5.png)

* Basic Example
    ```objectivec
     int* p = new int(1024);
     unique_ptr<int> ptr(p);
     int* second = new int(2048);
     ptr.reset(second);   //此处reset函数将会释放p的内存。
   //-----Below is WRONG!!! -------------------
   //  cout << *p << endl;
   //  *p = 1234;
   //   cout << *p << endl;
    ```
    
    * Source Code for reset
        ```objectivec
              void
              reset(pointer __p = pointer()) noexcept
              {
        	using std::swap;
        	swap(std::get<0>(_M_t), __p);   //交换两个指针。   
        	if (__p != pointer())   // 通过删除器释放原来指针所指向的空间。
        	  get_deleter()(__p);
              }
        ```
* release: 放弃对当前自由空间的控制权，需要通过删除器手动释放该空间。
    ```objectivec
    /// Release ownership of any stored pointer.
          pointer
          release() noexcept
          {
    	pointer __p = get();
    	std::get<0>(_M_t) = pointer();
    	return __p;
          }
    ```

* Example for smart pointer on an array.
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
