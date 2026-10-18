---
title: "题解 CF452F Permutation"
date: 2023-06-11T15:14:27+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-cf452f-permutation"
original: "https://www.luogu.com.cn/article/aitgm0ah"
math: true
---
好题。

考虑枚举每个点 $i$ 为等差数列中间的数，那么剩下两个数要在它一左一右且到 $i$ 距离相等。若将每个数都设一个 0/1 的状态，表示这个数字是否在 $i$ 左侧，那么若任意一对数都在 $i$ 的同侧，那么这两个数的状态同时为 $0/1$，最后也会形成一个以 $i$ 为中心的回文串。这个状态可以用线段树维护，是否为回文串可以在线段树上维护正向/反向的哈希值。
