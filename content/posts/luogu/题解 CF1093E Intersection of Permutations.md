---
title: "题解 CF1093E Intersection of Permutations"
date: 2023-06-26T20:57:42+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-cf1093e-intersection-of-permutations"
original: "https://www.luogu.com.cn/article/e6okfk6p"
math: true
---
考虑建立一个 $a$ 到 $b$ 的映射，设 $c_i$ 表示 $a_i$ 这个数在 $b$ 中的位置。那么查询就是求区间内 $c_i$ 在一个值域区间内数的个数，修改就是单点修改。

这里时间一维，下标一维，值域一维，是三维数点，用 CDQ 分治套树状数组即可，$O(n\log^2n)$。
