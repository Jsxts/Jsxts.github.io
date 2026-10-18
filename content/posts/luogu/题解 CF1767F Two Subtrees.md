---
title: "题解 CF1767F Two Subtrees"
date: 2023-04-25T21:00:30+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-cf1767f-two-subtrees"
original: "https://www.luogu.com.cn/article/fu9aee90"
math: true
---
学习了下高维莫队。

考虑普通莫队，是查询 $(l,r)$ 的答案，这里有两个指针在移动，是两维莫队。对于此题，有 $(l1,r1)$ 和 $(l2,r2)$，是四维莫队。

对于 $k$ 维莫队：取块长 $n^{\frac{k-1}{k}}$，复杂度为 $n^{\frac{2k-1}{2k}}$。

对于这道题，求众数不好删除，考虑在高维上的回滚莫队：

对于前三维，按询问在同一块内分类，对于第四维 $r2$，在前三维在同一块内时是递增的。这样，每次处理询问后分别将前三维的指针重置到块的边界，与普通的回滚莫队很相似。
