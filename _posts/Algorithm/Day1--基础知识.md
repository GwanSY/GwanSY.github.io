---
layout: post
title: "算法分析--基础知识"
date: 2024-11-25 11:29:08 +0800
categories: Algorithm
---
# 基础概念
1. 算法是将输入转换为输出的一系列计算步骤。
2. 算法5个重要特性：有穷性、确定性、可行性、输入、输出。
3. 运行时间对比表
```bash
lg(n) < sqrt(n) < n < n*lg(n) < n^2 < n^3 < 2^n < n!
```
# 伪代码当中的约定
1. 通过数组名[下标]的形式访问数组元素，伪代码中一般使用 1 作为第一个元素下标，但高级语言中一般使用 0 作为第一个元素下标。
2. 伪代码程序中的变量都是`局部变量`。
3. `:`表示子数组，A[i:j]表示数组A的一个子数组，由A[i],A[i+1]...A[j]组成。
4. 对象可由一组属性构成。
5. 伪代码中函数传参按照值传递。
6. 返回(return)语句执行后将返回到函数的调用点，大多数返回语句能返回一个值，伪代码中可以返回多个值。

# 渐近标记的定义
渐近上界`O`是随机情况不会超出的时间复杂度，
渐近下界`Ω`是随机情况至少需要的时间复杂度，
渐近紧确界`Θ`是渐近上界和渐近下界进行夹逼确定的时间复杂度。
## O 标记 (O-notation)
描述渐近上界 (asymptotic upper bound):
给定函数 \( g(n) \)，
\[
O(g(n)) = \{f(n): \exists c > 0, n_0 > 0, \forall n \geq n_0, 0 \leq f(n) \leq c g(n) \}.
\]

\( f(n) \in O(g(n)) \) 一般写为 \( f(n) = O(g(n)) \)，计算可转化为 \( f(n) \leq c g(n) \)。

## Ω 标记 (Ω-notation)
描述渐近下界 (asymptotic lower bound):
给定函数 \( g(n) \)，
\[
\Omega(g(n)) = \{f(n): \exists c > 0, n_0 > 0, \forall n \geq n_0, 0 \leq c g(n) \leq f(n) \}.
\]

\( f(n) \in \Omega(g(n)) \) 一般写为 \( f(n) = \Omega(g(n)) \)，计算可转化为 \( f(n) \geq c g(n) \)。

## Θ 标记 (Θ-notation)
描述渐近紧确界 (asymptotic tight bound):
给定函数 \( g(n) \)，
\[
\Theta(g(n)) = \{f(n): \exists c_1 > 0, c_2 > 0, n_0 > 0, \forall n \geq n_0, 0 \leq c_1 g(n) \leq f(n) \leq c_2 g(n) \}.
\]

\( f(n) \in \Theta(g(n)) \) 一般书写为 \( f(n) = \Theta(g(n)) \)，计算可转化为：
\[
c_1 g(n) \leq f(n) \leq c_2 g(n)
\]

