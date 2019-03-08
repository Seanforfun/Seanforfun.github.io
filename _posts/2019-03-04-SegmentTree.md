---
layout: post
title:  "Data structure | Segment Tree"
date:   2018-03-04 13:19
author: Botao Xiao
categories: DataStructure
comment: true
description: 
---

## Segment Tree
Segment tree is a kind of binary search tree. Segment tree is used to save nodes as a scope, and for each node, it has 2 childs(left and right node). 

### Example
1. Array
    ```
    0   1   2   3   4   5
    1   8   6   4   3   5
    ```

### Generation of Segment tree
![Imgur](https://i.imgur.com/6WEcMQ7.png)
    * Since segment tree is a complete binary tree, so if index of parent is k, we have:
        * left = k << 1
        * right = k << 1 | 1
    * Break of spliting a segment node: left index == right index.
    * Assume we want create a segment tree to save the max value for each segment.
    ```Java
    public class SegmentTree {

        private int[] segments; //Array to save segment tree.
        private int[] num;
        public SegmentTree(int[] arr){
            this.num = arr;
            this.segments = new int[arr.length << 2 + 1];
            build(1, 1, arr.length);
        }

        /**
         * Starts from build(1, 1, n)
         * @param k index of current node in array
         * @param l left index of current node, representing the left boundary of the segment.
         * @param r right index of current node, representing the right boundary of the segment.
         */
        private void build(int k, int l, int r){
            if(l == r){ // leaf node, cannot be splited any more, assign value.
                segments[k] = num[l - 1];
            }else{
                int m = l + ((r - l) >> 1);
                build(k << 1, l, m); // Build left child
                build(k << 1 | 1, m + 1, r); // Build right child
                segments[k] = Math.max(segments[k << 1], segments[k << 1 | 1]);
            }
        }
    }
    ```

### Operations of segment tree
1. Update segment tree. For example, we want to update value at index 3, we call it a[3] += 7. We need to update all nodes that contains this array, from its leaf to root. which means we can update the value in the direction of root to leaf, and the path is unique.
![Imgur](https://i.imgur.com/XAYZ1jJ.png)
```Java
 /**
 * Since we will update from root node of the segment tree.
 * We call this method recursively from update(p, diff, 1, length of the num array, 1)
 * @param p index of the number to update in array num.
 * @param diff new different to be added in the num array at index p.
 * @param l left index of current node, representing the left boundary of the segment.
 * @param r right index of current node, representing the right boundary of the segment.
 * @param k the node index in segment tree.
 */
public void update(int p, int diff, int l, int r, int k){
    if(l == r){ // l == r means current node is a leaf node, we just need to update its value.
        segments[k] += diff;
        num[p] += diff;
    }else{  // we need to update the value of current node and one of its child.
        int m = l + ((r - l)>>1);
        if((p + 1) <= m){ // need to update left child.
            //recursively update its child.
            update(p, diff, l, m, k << 1);
        }else{  // need to update right child.
            update(p, diff, m + 1, r, k << 1 | 1);
        }
        segments[k] = Math.max(segments[k << 1], segments[k << 1 | 1]);
    }
}
```

2. Query the segment tree. Use divide and conquer method to solve this question.
```Java
/**
 * This function returns the max value in scope [L, R].
 * This function is initially called with query(L, R, 1, array length of num, 1).
 * @param L left boundary of scope to be searched(Included).
 * @param R right boundary of scope to be searched(Included).
 * @param l for current method calling, the left scope in segment tree.
 * @param r for current method calling, the right scope in segment tree.
 * @param k index of current node in segment tree.
 * @return returns the max value of in scope [L, R]
 */
public int query(int L, int R, int l, int r, int k){
    if(l >= L && r <= R){ 
        //segment tree region is included in the searching region so we don't need to recursively call this method at this point.
        return segments[k];
    } else{
        int max = Integer.MIN_VALUE;
        int m = l + ((r - l) >> 1);
        if(L <= m){ 
            // L <= m means there are some parts of the query region contain in the left child, 
            //we need to recursively call the left child for getting the max value for [L, m]
            max = Math.max(max, query(L, R, l, m, k << 1));
        }
        if(R > m){
            // Similar as previous one but we are checking the value for scope [m + 1, R]
            max = Math.max(max, query(L, R, m + 1, r, k << 1 | 1));
        }
        return max;
    }
}
```

3. Lazy Region update
```Java
/**
 * @param L left boundary of scope to be updated.
 * @param R right boundary of scope to be updated.
 * @param diff  diff to be added.
 * @param l left boundary of current node.
 * @param r right boundary of current node.
 * @param k index of current in segment tree.
 */
public void updateLazy(int L, int R, int diff, int l, int r, int k){
    if(l >= L && r <= R){
        //Since we need to update all values in this region and current region is included in the node,
        // We need to update the max values saved in this node.
        // Meanwhile we update the lazy array.
        lazy[k] += diff;
        segments[k] += diff;
    }else{
        pushDown(k);
        int m = l + ((r - l) >> 1);
        if(L <= m){
            updateLazy(L, R, diff, l, m, k << 1);
        }
        if(R > m){
            updateLazy(L, R, diff, m + 1, r, k << 1 | 1);
        }
        segments[k] = Math.max(segments[k << 1], segments[k << 1 | 1]);
    }
}
private void pushDown(int k){
    if(lazy[k] > 0){    // if lazy, we need to update current value.
        lazy[k << 1] += lazy[k];
        lazy[k << 1 | 1] += lazy[k];
        segments[k << 1] += lazy[k];
        segments[k << 1 | 1] += lazy[k];
        lazy[k] = 0;
    }
}
```

### Reference
1. [线段树详解](https://www.cnblogs.com/xenny/p/9801703.html)