---
title: "题解 [ABC230G] GCD Permutation"
date: 2023-06-18T18:43:39+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-abc230g-gcd-permutation"
original: "https://www.luogu.com.cn/article/g910s5a5"
math: true
---
以后看到 GCD 要想想推式子。

将最终答案除以 $2$，就变为无序。

见到两维都有 GCD 的限制，考虑枚举下标的 GCD，设 $f_d$ 为 $\gcd(i,j)=d$ 的答案，求出 $g(d)=\sum_{d|i}f_i$，再用狄利克雷后缀和倒推即可。

考虑枚举 $d$，现在可用的数为 $d$ 的倍数，记这些数为 $a$ 数组，再容斥一下，要求的就变为

$$\sum_{i=1}^n\sum_{j=1}^n [\gcd(a_i,a_j)=1]$$

记 $c_i$ 为 $a$ 数组中 $i$ 的出现次数，则：

$$=\sum_{i=1}^n\sum_{j=1}^n c_ic_j[\gcd(i,j)=1]$$
$$=\sum_{i=1}^n\sum_{j=1}^n c_ic_j\sum_{x|i,x|j}\mu(x)$$
$$=\sum_{x=1}^n\mu(x)\sum_{i=1}^{n/x}c_{ix}\sum_{j=1}^{n/x} c_{jx}$$
$$=\sum_{x=1}^n\mu(x)\left(\sum_{i=1}^{n/x}c_{ix}\right)^2$$

考虑每次逐个加入算贡献，加入一个 $x$ 每个 $x$ 的因数 $y$ 贡献增加 $(2c_y+1)\times \mu(y)$，直接维护因子集合即可。

注意虽然将答案变为了无序，但是这题可以选择两个相同的位置，而这些位置又只被算了一次，所以要将 $g$ 加上 $a$ 中不为 $1$ 的数个数。

时间复杂度 $O(n\log n)$ 乘上因子个数的最大值。
