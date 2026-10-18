---
title: "题解 CF1067D Computer Game"
date: 2023-04-30T15:26:45+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-cf1067d-computer-game"
original: "https://www.luogu.com.cn/article/z1fp8wvm"
math: true
---
设总时间为 $T$。

首先考虑升级肯定挑 $p_ib_i$ 最大的升，并且升完后一定一直玩这个游戏。

现在考虑升级前的决策，设 $f_t$ 表示还剩 $t$ 秒还没开始刷 $b_ip_i$ 最大的游戏时，期望最大收益。

考虑当前秒是否能升级，设 $W=\max_{i} b_ip_i$，列出如下转移式：

$$f_t=\max_{i}\{p_i\times(a_i+W\times(t-1))+(1-p_i)\times f_{t-1}\}$$
$$=\max_{i}\{p_i\times(a_i+W\times(t-1)-f_{t-1})\}+f_{t-1}$$
$$=f_{t-1}+\max_{i}\{p_i\times(W\times(t-1)-f_{t-1})+p_ia_i\}$$

设 $k_t=Wt-f_t$，显然 $k_t>0$。

看到后面的式子，想到斜率优化，把 $(p_i,p_ia_i)$ 看成坐标系上的一个点，那么就是一条斜率为 $-k_t$ 的直线过一点使得截距最大。这个点肯定在上凸包上，维护凸包再二分到切点的位置，时间复杂度 $T\log n$。

继续优化。观察 $k_t$ 的性质，发现 $k_t$ 不降，证明：$k_t-k_{t-1}=f_{t-1}-f_t+W$，据定义显然 $f_t-f_{t-1}$ 要不大于 $W$，得证。

由上面 $f$ 的转移，可得出 $k$ 的转移为 $k_t=k_{t-1}+W-\max_{i}\{p_i\times k_{t-1}+p_ia_i\}$，当 $i$ 确定时 $k_t=(1-p_i)\times k_{t-1}+W-p_ia_i$，是一个一次函数，可以快速从 $k_t$ 得到 $k_{t+j}$。

既然 $k_t$ 不降，那么对应在凸包上的切点的 $x$ 坐标不降，考虑二分求出凸包上每个点作为切点的时间区间，然后就可以求出对应的 $k$（相当于同一个一次函数复合若干次），时间复杂度 $O(n\log T)$。
