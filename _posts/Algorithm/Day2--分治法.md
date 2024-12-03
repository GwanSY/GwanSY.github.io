---
layout: post
title: "算法分析--分治法"
date: 2024-11-27 11:29:08 +0800
categories: Algorithm
---
# Divide and Conquer 分治法 
把一片领土分解成若干小块儿，然后一块儿一块儿分解。
1. Divide the problem 原规模的n变小
2. Conquer 递归的解每个小问题
3. Commbine 将子问题的解合并为原问题的解。

## Merge sort 归并排序 Θ(nlgn)————分治法
### 递归式
$$
T(n)=
\left\{\begin{matrix}
Θ(1),n=1 \\
2T(n/2)+Θ(n) n>1
\end{matrix}}\right.
$$
### 伪代码

Merge-SORT(A, n) or SORTS A[1,n]
1. If n-1, done
2. 拆分输入A[1,...,[n/2]] and A[[n/2]+1,...,n] (向上取整)
3. (关键！)把排好序的两个表归并
    a. 将待排序数组一分为二

    b. 递归的对每一个子数组进行排序

    c. 合并 O(n)：两张已排序的列表中，最小元素是多少；将两个中较小的输出到最终的输出数组中；继续比较两个中最小的数，然后继续输出到最终数组中...
    
    【注意：】每一步都是固定数目的操作，和数组尺寸无关

```bash
MERGE-SORT(A, p, r)
    if p < r
        q = (p + r) / 2          // 计算中间索引
        MERGE-SORT(A, p, q)         // 递归排序左半部分
        MERGE-SORT(A, q + 1, r)    // 递归排序右半部分
        MERGE(A, p, q, r)        // 合并已排序的两部分

MERGE(A, p, q, r)
    n1 = q - p + 1                 // 左半部分的元素个数
    n2 = r - q                    // 右半部分的元素个数
    Let L[0:n1-1] and R[0:n2-1] be new array   // 创建临时数组L和R

    for i = 1 to n1            // 将左半部分复制到临时数组L
        L[i] = A[p + i - 1]
    for j = 1 to n2            // 将右半部分复制到临时数组R
        R[j] = A[q + j]

    i = 1
    j = 1
    for k = p to r                // 合并L和R到原数组A
        if L[i] <= R[j]
            A[k] = L[i]
            i = i + 1
        else
            A[k] = R[j]
            j = j + 1

```

### java实现
```bash
public class MergeSort {
    
    // 主排序方法
    public static void mergeSort(int[] arr) {
        if (arr.length > 1) {
            // 找到数组的中间点
            int q = arr.length / 2;
            
            // 分割数组为两半
            int[] left = new int[q];
            int[] right = new int[arr.length - q];
            
            System.arraycopy(arr, 0, left, 0, q);
            System.arraycopy(arr, q, right, 0, arr.length - q);
            
            // 递归排序左右两部分
            mergeSort(left);
            mergeSort(right);
            
            // 合并排序后的数组
            merge(arr, left, right);
        }
    }
    
    // 合并两部分
    private static void merge(int[] arr, int[] left, int[] right) {
        int i = 0, j = 0, k = 0;
        
        // 比较并合并左右数组的元素
        while (i < left.length && j < right.length) {
            if (left[i] < right[j]) {
                arr[k++] = left[i++];
            } else {
                arr[k++] = right[j++];
            }
        }
        
        // 将左边剩余的元素加入
        while (i < left.length) {
            arr[k++] = left[i++];
        }
        
        // 将右边剩余的元素加入
        while (j < right.length) {
            arr[k++] = right[j++];
        }
    }

    // 测试归并排序
    public static void main(String[] args) {
        int[] arr = { 38, 27, 43, 3, 9, 82, 10 };
        mergeSort(arr);
        
        // 输出排序后的数组
        for (int num : arr) {
            System.out.print(num + " ");
        }
    }
}
```

## Binary search 二分查找法Θ(logn)
设一个数为x，在一个已排序的数组中找到这个x
$$T(n) = T(n/2)+Θ(1) = Θ(logn)$$
1. Divide the problem : 把x与数组的中间元素相比较
2. Conquer ：判断x与中间元素的大小，选择左边还是右边进行递归判断
3. Commbine 
### 伪代码

```bash
function binarySearch(array, target):
    p = 0
    r = length(array) - 1
    while p <= r:
        q = p + (r - p) / 2
        if array[q] == target:
            return q
        else if array[q] < target:
            p = q + 1
        else:
            r = q - 1
    return -1  // target not found
```

### JAVA实现

```bash
public class BinarySearch {
    public static int binarySearch(int[] array, int target) {
        int p = 0;
        int r = array.length - 1;
        
        while (p <= r) {
            int q = p + (r - p) / 2;
            
            if (array[q] == target) {
                return q;
            } else if (array[q] < target) {
                p = q + 1;
            } else {
                r = q - 1;
            }
        }
        
        return -1;  // target not found
    }
    
    public static void main(String[] args) {
        int[] array = {1, 3, 5, 7, 9, 11, 13};
        int target = 7;
        int result = binarySearch(array, target);
        System.out.println(result);  // Output: 3
    }
}
```

## Powering number 乘方问题Θ(lgn)
给一个实数或者浮点数x,再给一个正整数n,计算 $x^n$.

当n为偶数时：$x^n = x^(n/2)·x^(n/2)$
当n为奇数时：$x^n = x^((n-1)/2)·x^((n-1)/2)·x$

$T(n)=T(n/2)+Θ(1)=Θ(lgn)$
### 伪代码

```bash
function power(x, n):
    if n == 0:
        return 1  // x^0 = 1
    if n is even:
        half = power(x, n / 2)
        return half * half
    else:
        return x * power(x, n - 1)
```
### JAVA实现

```bash
public class Power {
    public static double power(double x, int n) {
        if (n == 0) {
            return 1;  // x^0 = 1
        }
        
        // Handle negative exponent
        if (n < 0) {
            x = 1 / x;
            n = -n;
        }
        
        // Fast exponentiation
        double result = 1;
        while (n > 0) {
            if (n % 2 == 1) {
                result *= x;  // If n is odd
            }
            x *= x;  // Square the base
            n /= 2;   // Halve the exponent
        }
        
        return result;
    }
    
    public static void main(String[] args) {
        double x = 2;
        int n = 10;
        System.out.println(power(x, n));  // Output: 1024.0
    }
}
```
## Fibonacci numbers 斐波那契数

 