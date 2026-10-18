---
title: "题解 CF1320D Reachable Strings"
date: 2023-06-12T13:18:42+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-cf1320d-reachable-strings"
original: "https://www.luogu.com.cn/article/u94fek0s"
math: true
---
当一个东西要通过某些操作变成另一个东西时，若操作可逆，则可以看成将两个东西同时操作成一个特定的形式判断是否相等。

对于这题，操作可以视为将 $11$ 随意地移动，那么如果将两个串的 $11$ 都移到最后，就可以视为删掉它们，判断剩下的部分是否相等，用线段树维护区间删掉所有连续的 $11$ 后的哈希值即可。
