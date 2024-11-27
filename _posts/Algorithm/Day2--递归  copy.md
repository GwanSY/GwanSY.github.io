---
layout: post
title: "算法分析--递归"
date: 2024-11-26 11:29:08 +0800
categories: Algorithm
---
# Recursion-tree method 递归树法
【注意：】该方法不够严谨需要验证。
在递归树中，每个结点表示一个单一子问题的代价，子问题对应某次递归函数的调用。我们将树中每层中的代价求和，得到每层代价，然后将所有层的代价求和，得到所有层次递归调用的总代价。
$$
T(n)=T(n/4)+T(n/2)+Θ(sqrt(n))
T(n)=T(n/4)+T(n/2)+c(sqrt(n))
$$

![](https://cdn.jsdelivr.net/gh/Country-If/Typora-images/img/202401041357645.png)
最左边的分枝为从根结点到叶结点的路径，在所有从根结点到叶结点的路径中最短，对应子问题数据规模为
 