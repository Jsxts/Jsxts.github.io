---
title: "题解 CF961G Partitions"
date: 2023-09-07T14:57:52+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-cf961g-partitions"
original: "https://www.luogu.com.cn/article/875pp5cf"
math: true
---
首先发现每个人的贡献次数肯定是一样的。

所以只考虑第一个人的贡献次数，枚举它在的集合大小，得到如下式子：

$$\sum_{j=1}^{n-k+1}j\dbinom{n-1}{j-1}\begin{Bmatrix}n-j\\k-1\end{Bmatrix}$$

对其他人也一样。然后套用第二类斯特林数·列求出一列的斯特林数即可（也可以展开之后推下去）。

但是要任模，很麻烦。

换个思路，因为贡献乘上了一个区间长度，我们可以发现如果加进来你的集合，就会对你有一次的贡献。那么自己对自己有 $\begin{Bmatrix}n\\k\end{Bmatrix}$ 的贡献，每个别人对自己就是钦定它们绑在一起再划分，共 $(n-1)\times \begin{Bmatrix}n-1\\k\end{Bmatrix}$。

对每个人贡献相同，最后乘上 $\sum w_i$ 即可。
