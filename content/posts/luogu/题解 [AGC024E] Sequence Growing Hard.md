---
title: "题解 [AGC024E] Sequence Growing Hard"
date: 2023-06-30T19:19:40+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-agc024e-sequence-growing-hard"
original: "https://www.luogu.com.cn/article/yn0kwol7"
math: true
---
第一眼是 DP，但不知道如何设计状态，考虑先找性质。

将从 $A_0$ 加数到 $A_n$ 变成从 $A_n$ 删数到 $A_0$。考虑删掉一段同样的数中任意一个效果一样，会算重，于是钦定每次只能删同一段数的最右边。发现对于字符串 $s$，若删掉的位置是 $k$，则需要满足起始位置为 $k$ 的后缀字典序大于 $k+1$ 的后缀。由于删的是最右边，所以 $s_k\neq s_{k+1}$，所以只需满足 $s_k>s_{k+1}$ 即可。

现在可以开始 DP 了。设 $f_{i,j}$ 为前 $i$ 个序列，使用了值域在 $[1,j]$ 的数的方案数。我们发现整个序列中 $1$ 较为特殊，考虑枚举 $1$ 出现的第一个位置，设为 $k$，这样在 $k$ 之后 $>1$ 的数必须先删完再删 $1$，而 $1$ 是从右往左删的，所以 $k$ 是最后一个被删掉的。由于 $[2,j]$ 和 $[1,j-1]$ 没有区别，于是前面就是 $f_{k-1,j-1}$。再枚举 $k$ 何时被删，可以列出转移方程：

$$f_{i,j} = \sum_{k = 1}^i\sum_{l=i-k+1}^if_{k-1,j-1}f_{i-k,j}\dbinom{l - 1}{i - k}$$
$$= \sum_{k = 1}^if_{k-1,j-1}f_{i-k,j}\sum_{l=i-k+1}^i\dbinom{l - 1}{i - k}$$

直接预处理 $S_{i,k}=\sum_{l=i - k + 1}^i\dbinom{l - 1}{i - k}$ 即可，注意 $f_{i,j}$ 要加上 $f_{i,j-1}$ 的方案数。

$O(n^3)$。
