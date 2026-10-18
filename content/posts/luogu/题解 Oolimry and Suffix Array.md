---
title: "题解 Oolimry and Suffix Array"
date: 2023-03-20T13:08:30+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-oolimry-and-suffix-array"
original: "https://www.luogu.com.cn/article/3nsy7ow3"
math: true
---
如果假设每个字符都不能相等，那么给出的 sa 数组对应的就是字符串中 $n$ 个字母的大小关系。现在可以相等，那么我们可以对于每个位置考虑是否能与前面一位相等。

考虑原序列 sa 为 $[3,2,4,1,0,5,6]$，定义 $rk_{sa_i}=i$。现在考虑 $s_4$ 能否与 $s_2$ 相等。显然按照规则，这一位相等就比较下一位，这样就相当于比较 $rk_3$ 与 $rk_5$ 的大小，发现 $rk_3$ 大，于是可以相等。

这样我们就得到若干个位置能与上一位相等，考虑计数，枚举有几个位置要相等，答案为

$$\sum_{i=0}^m\dbinom{k}{n-i}\dbinom{m}{i}=\dbinom{m+k}{n}$$
