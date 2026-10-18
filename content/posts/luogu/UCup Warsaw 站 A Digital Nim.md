---
title: "UCup Warsaw 站 A Digital Nim"
date: 2024-03-15T22:43:08+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "ucup-warsaw-站-a-digital-nim"
original: "https://www.luogu.com.cn/article/dtmeujsx"
math: true
---
妙妙题。

令 $x$ 的数位和为 $ds(x)$，$<x$ 的数中最大的败态为 $x-last(x)$。那么，若 $last(x)> ds(x)$，$last(x+1)=1$，否则 $last(x+1)=last(x)+1$。

我们只需判断 $last(n+1)$ 是否为 $1$。

不妨考虑一个数慢慢累加，从 $0$ 加到 $n+1$，每次加上最大的 $10^i$ 满足加上后还是 $\le n+1$，然后动态维护 $last(x)$。

那么这样的过程可以看成一个过程：令 $x\to x+10^i$，满足 $x\equiv0\pmod {10^i}$。

发现我们不关心 $x$ 的具体值，只需用 $last(x),ds(x)$ 即可转移，即：设 $dp_{i,j,k}$ 表示要执行 $x\to x+10^i$，满足该位为 $x$ 的最低非零位且加上后不会进位，$ds(x)=j,last(x)=k$ 时 $last(x+10^i)$ 会为何值。

考虑转移，发现一次 $x\to x+10^i$ 可以分解成 $x\to x+10^{i-1},x+10^{i-1}\to x+2\times 10^{i-1},\dots$，而中间过程的数位和又是容易知道的，所以可以转移了！

由于 $ds(x),last(x)$ 都是 $O(d \log_{d}n)$ 级别的（$last(x)$ 超过数位和的最大上界可以不用转移），所以总复杂度为 $O(d^3\log_d^3n)$。
