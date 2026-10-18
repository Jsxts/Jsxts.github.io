---
title: "题解 CF1770F Koxia and Sequence"
date: 2023-04-09T15:38:13+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-cf1770f-koxia-and-sequence"
original: "https://www.luogu.com.cn/article/q95mthfz"
math: true
---
好题。

发现这个 $a_i$ 的顺序不重要，因此对于每个位置，上面所有可能的数的异或和是相同的，那么如果 $n$ 为偶数，答案就是  $0$，$n$ 为奇数就是第一个位置上面所有可能的数的异或和。

按位考虑，对于第 $p$ 位计算这位为 $1$ 的方案数。

全部或起来等于 $y$，这个可以容斥，设 $f(t)$ 为钦定全部或起来的值 $s \subseteq t$ 的方案数，则 $ans=\sum_{t\subseteq y}-1^{|y|-|t|}f(t)$。

考虑计算 $f(t)$，枚举每一个数，使其和为 $x$：

$$f(t)=\sum_{\sum a_i=x-2^p}[a_1\subseteq t-2^p]\prod_{i=2}^n [a_i \subseteq t]$$

此时，一个非常巧妙的方法是将 $[a_i \subseteq t]$ 通过组合数奇偶性判断的方法转换为 $\dbinom{t}{a_i}\bmod 2$。

如何证明 $\dbinom{n}{m}\bmod 2=[m \subseteq n]$ 呢？

判断一个组合数 $\dbinom{n}{m}$ 的奇偶性，要看其中质因子 $2$ 的数量，而 $\dbinom{n}{m}=\dfrac{n!}{m!(n-m)!}$，我们考虑计算 $n!$ 中含有质因子 $2$ 的个数，为：

$$f(n)=\sum_{i=1}^{inf}\lfloor\dfrac{n}{2^i}\rfloor$$

设 $n=\sum_{j}2^{a_j}$，其中 $a_i$ 互不相同，则：

$$\lfloor\dfrac{n}{2^i}\rfloor=\sum_{j}\lfloor\dfrac{2^{a_j}}{2^i}\rfloor$$

因为当 $a_j\ge i$ 时无影响， $a_j<i$ 时所有的总和也不会超过 $1$，都当成 $0$ 算没问题。

$$f(n)=\sum_{i=1}^{inf}\sum_{j}\lfloor\dfrac{2^{a_j}}{2^i}\rfloor$$
$$=\sum_{j}\sum_{i=1}^{inf}\lfloor2^{a_j-i}\rfloor$$
$$=\sum_{j}2^{a_j}-1$$
$$=n-p(n)$$

其中 $p(n)$ 为 $n$ 二进制表示中 $1$ 的个数。

那么如果 $\dbinom{n}{m}$ 为奇，即不含质因子 $2$，必须要有 $f(n)=f(m)+f(n-m)$，即 $n-p(n)=m-p(m)+n-m+p(n-m)$。

那么 $p(n)=p(m)+p(n-m)$。显然两个数加起来进位的话，$1$ 的个数会减少，所以 $m \& (n-m)=0$，即 $m\&n=m$，$m\subseteq n$。

有了这个，又因为只关心 $f(t)$ 的奇偶性，原式可以转换为

$$f(t)=\left[\sum_{\sum a_i=x-2^p}\dbinom{t-2^p}{a_1}\prod_{i=2}^n \dbinom{t}{a_i}\right]\bmod 2$$
这是范德蒙德卷积的形式，可以写成

$$=\dbinom{nt-2^p}{x-2^p}\bmod 2$$
$$=[x-2^p\subseteq nt-2^p]$$

可以枚举 $p$ 后枚举 $y$ 的子集计算，时间复杂度 $O(y\log y)$。
