---
title: "Solution of ARC 140"
date: 2022-05-26T14:09:00+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "solution-of-arc-140"
original: "https://www.luogu.com.cn/article/9g3o2lr6"
math: true
---
[比赛链接](https://atcoder.jp/contests/arc140)

## A

$f$ 函数其实就是求的是最小周期的长度（一定要都是完整的周期），所以可以枚举周期长度，之后对于周期的每一位贪就行了。

算法：贪心，字符串 *1600

## B

除了 `A..ARC..C` 这种形式的字符串，其它的都没有贡献。

考虑把这些字符串都提取出来，设第 $i$ 个字符串的权值 $x_i=min(\text{the number of A},\text{the number of C})$，发现在奇数次操作最优是把 $x_i\ge2$ 中的最小的 $x$ 减一，偶数次操作就是删掉 $x$ 最小的字符串。

容易发现这样操作的答案就是 $\min(2\times tot,\sum x_i)$，其中 $tot$ 表示这些字符串的个数。

算法：构造 *1900
## C

首先如果没有第一个数的限制，可以构造一种最优方案（假设 $n$ 为奇数，偶数类似）：

设 $x = \lfloor \frac{n+1}{2}\rfloor$，则：

$$P=\{x,x+1,x-1,x+2,x-2,\cdots,n,1\}$$

可以构造出长为 $n-1$ 的最长上升子序列。

假设第一个数为 $y$，设 $y=x+k$。

则可以用类似的方法构造出长为 $n-2$ 的最长上升子序列：

$$P=\{y,x,x-1,x+1,x-2,x+2,\cdots,x-k,x+k+1,x-(k+2),\cdots,n,1\}$$

这是 $k>0$ 的构造方法，$<0$ 类似。

可以证明这是最优解（？

算法：构造 *2000

## D

已经建好的边把点划分成若干个基环树和树。

- 如果是基环树，那么这个连通块没有未定的边，直接算贡献。贡献为 $a\times n^b$，其中 $a$ 是基环树的数量，$b$ 是树的数量。

- 如果是树，那么有且仅有一条未定的边。

设 $f_{i,j}$ 为前 $i$ 棵树，选 $j$ 棵组成一个连通块的方案数，转移方程为

$$f_{i,j}=f_{i-1,j}+f_{i-1,j-1}\times sz_i$$

设共有 $k$ 棵树，最后答案为

$$\sum_{i=1}^kf_{k,i}\times(i-1)!\times n^{k-i}$$

因为每棵树是不同的，并且剩下的 $k-i$ 棵树可以随便连，所以要乘上 $(i-1)!\times n^{k-i}$。

算法：DP，组合数学 *2200
## E

在 $(i,j)$ 填入 $(\lfloor \frac{i}{p}\rfloor\lfloor\frac{j}{p}\rfloor+i+j)\bmod p+1$ 即可，其中 $p=23$。

证明：考虑对于一个素数 $p$，用 $0 \sim p-1$ 构造一个 $p^2 \times p^2$ 的满足条件的矩阵。

先构造 $p$ 个 $p \times p$ 的满足条件的矩阵 $B_{0,1,\cdots.p-1}$，再用这些矩阵构造出答案。对于第 $k$ 个矩阵，令 $B_k(i,j)=(i+j+k)\bmod p$，这样每个小矩阵每行每列都没有两个相同的数。

那么现在我们要用 $p \times p$ 个小矩阵来构造答案，令在 $(i,j)$ 位置上的矩阵为 $B_{ij\bmod p}$，构造出来的矩阵就可以满足条件。

证明：假设矩形四个角所在的小矩阵分别为 $B_a,B_b,B_c,B_d$（从左上角开始，顺时针），因为每个小矩阵每行每列都没有两个相同的数，所以这四个角如果在一个或两个矩阵里都必定满足条件。

如果四个角在四个不同的矩阵，那么四个数相同当且仅当 $a-d=b-c$，这个可以结合 $B$ 的构造方法理解。但由于 $(i,j)$ 上是 $B_{ij\bmod p}$，所以 $a-d \ne b-c$。

证毕。这样当 $p=23$ 时，可以构造出 $529 \times 529$ 的矩阵，输出左上角的 $n \times m$ 矩阵即可。

---
其实还有更简单的证明方法。

对于同列相等的两个数，即 $a_{x,j}=a_{y,j}(x<y)$，可得如下式子：

$$\lfloor \frac{x}{p}\rfloor\lfloor\frac{j}{p}\rfloor+x+j\equiv\lfloor \frac{y}{p}\rfloor\lfloor\frac{j}{p}\rfloor+y+j\pmod{p}$$

$$(\lfloor \frac{x}{p}\rfloor-\lfloor \frac{y}{p}\rfloor)\lfloor\frac{j}{p}\rfloor\equiv y-x \pmod{p}$$

如果 $\lfloor \frac{x}{p}\rfloor\equiv\lfloor \frac{y}{p}\rfloor \pmod p$，那么 $\lfloor \frac{x}{p}\rfloor+kp=\lfloor \frac{y}{p}\rfloor$。因为 $p^2>y$，所以 $k$ 只可能为 $0$， 即 $\lfloor \frac{x}{p}\rfloor=\lfloor \frac{y}{p}\rfloor$，所以 $x=y$，与 $x<y$ 矛盾。

因为 $p$ 为质数，所以一定存在 $(\lfloor \frac{x}{p}\rfloor-\lfloor \frac{y}{p}\rfloor) \bmod p$ 的逆元，所以上式可变为

$$\lfloor\frac{j}{p}\rfloor\equiv \dfrac{y-x}{\lfloor \frac{x}{p}\rfloor-\lfloor \frac{y}{p}\rfloor} \pmod{p}$$

此时如果存在两个不同的 $j$ 使得 $\lfloor\frac{j_1}{p}\rfloor\equiv\lfloor\frac{j_2}{p}\rfloor\equiv\dfrac{y-x}{\lfloor \frac{x}{p}\rfloor-\lfloor \frac{y}{p}\rfloor} \pmod{p}$，同上得 $j_1=j_2$，矛盾。

证毕。

算法：数学、构造 *3600

## F

算法：分治FFT、数学、构造 *3300
