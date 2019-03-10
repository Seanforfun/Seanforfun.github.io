---
layout: post
title:  "Data structure | Binary Index Tree"
date:   2018-03-03 19:18
author: Botao Xiao
categories: DataStructure
comment: true
description: 
---

Binary tree is used to save the prefix sum of a array, it is not linear so and the O time complexity is O(lgN).

### Structure of Binary Index Tree
![Imgur](https://i.imgur.com/n8TQjO5.png)
C array is the binary index tree, A array is the element array.
C[1]: 001, ending by 0 0s, v = 1 - 2^0 + 1, C[1] = A[1]  
C[2]: 010, ending by 1 0, v = 2 - 2 ^ 1 + 1, C[2] = A[2]
C[3]: 101, ending by 0 0, v = 3 - 2^0 + 1 = 3, C[3] = A[1] +A[2] + A[3]
C[4]: 100, ending by 2 0s, v = 4- 2^2 + 1 = 1, C[4] = A[1] + A[2] + A[3] + A[4]
v =  index - 2 ^ (number of zeros at the end of the binary value) + 1.

### Implementation of Binary index tree
1. Structure
```
index  1  2  3  4  5  6  7  8
A      2  3  5  7  2  5  1  5
```

2. Calculate the low bit value, which is 2 ^ n in previous expressions
```Java
private int lowBit(int n){
    return n & (-n);
}
```

3. Create the binaryIndexTree
```Java
public int[] c;
public int[] num;
public BinaryIndexTree(int[] arr){
        c = new int[arr.length + 1];
        num = arr;
        for(int i = 1; i <= arr.length; i++){
            int lowBit = lowBit(i);
            System.out.println("-------------------");
            System.out.println(lowBit);
            int sum = 0;
            for(int j = i - lowBit(i) + 1; j <= i; j++){
                sum += arr[j - 1];
            }
            c[i] = sum;
        }
    }
```

4. Calculate the sum of previous n terms
```Java
public int sum(int n){
        int sum = 0;
        while(n > 0){
            sum += c[n];
            n -= lowBit(n);
        }
        return sum;
    }
```

5. Update the tree
```Java
public void update(int i, int val){
	int diff = val - num[i];
	this.num[i - 1] = val;
	while(i <= this.num.length){
		c[i] += diff;
		i += lowBit(i);
	}
}
```

### Reference
1. [(LeetCode 307) Range Sum Query - Mutable(树状数组讲解)](https://blog.csdn.net/dreamgchuan/article/details/51173561)