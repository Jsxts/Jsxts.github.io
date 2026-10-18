---
title: "题解 CF475E Strongly Connected City 2"
date: 2023-04-26T22:14:29+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-cf475e-strongly-connected-city-2"
original: "https://www.luogu.com.cn/article/3hlwcxsc"
math: true
---
首先很明显，对于一个边双可以将边定向为若干环，使其中的点都能两两到达。之后将边双缩起来，变成了一棵树。直接 dp 不好做，考虑找这个构造的性质：

对于一个点，若以其为分界，则必定有一部分子树的边全为 $fa\to son$，一部分子树的边全为 $son\to fa$。证明可以考虑反证法，若有反向的边形成的连通块，使其全部正过来不会减少答案。

那么的话，对于这个点，若以他它为根，指向它的儿子的子树大小和为 $sz$，树的大小为 $m$，那么点对数即为 $sz\times (m - sz - 1)=-sz^2+(m-1)sz$，当 $sz$ 取最接近 $\frac{m-1}{2}$ 时答案最大，一个类似背包的 DP 就可解决。
