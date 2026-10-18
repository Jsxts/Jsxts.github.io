---
title: "题解 [ABC233Ex] Manhattan Christmas Tree"
date: 2023-06-28T10:42:49+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-abc233ex-manhattan-christmas-tree"
original: "https://www.luogu.com.cn/article/yorxyair"
math: true
---
欧几里得距离：$\sqrt{(x_i-x_j)^2+(y_i-y_j)^2}$

曼哈顿距离：$|x_i-x_j|+|y_i-y_j|$

切比雪夫距离：$\max(|x_i-x_j|,|y_i-y_j|)$

其中若设 $A(x_i+y_i,x_i-y_i),B(x_j+y_j,x_j-y_j)$，那么 $A$ 与 $B$ 的切比雪夫距离就是原来 $i,j$ 的曼哈顿距离。

这有什么用呢？

在这题，二分答案后变为求与 $(x,y)$ 距离 $\le mid$ 的点的数量，考虑转换成切比雪夫距离，那么要求 $\max(|x_i-x|,|y_i-y|)\le mid$，也就是在以 $(x,y)$ 为中心边长为 $2mid$ 的正方形内部，就是二维数点，直接用主席树维护即可。

$O(n\log^2n)$。
