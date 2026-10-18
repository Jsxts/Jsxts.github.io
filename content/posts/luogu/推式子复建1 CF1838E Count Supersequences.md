---
title: "推式子复建1 CF1838E Count Supersequences"
date: 2024-03-07T12:46:02+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "推式子复建1-cf1838e-count-supersequences"
original: "https://www.luogu.com.cn/article/igobi5st"
math: true
---
容易得到：

$$\sum_{i=0}^mk^i\dbinom{m - i+n-1}{n-1}$$

对 $n$ 递推：

$$f(n+1)=\sum_{i=0}^mk^i\dbinom{m - i+n}{n}$$

$$=\sum_{i=0}^mk^i\dbinom{m - i+n-1}{n}+\sum_{i=0}^mk^i\dbinom{m - i+n-1}{n-1}$$

$$=\sum_{i=1}^{m}k^{i-1}\dbinom{m - i+n}{n}+f(n)$$

$$=\dfrac{1}{k}f(n+1)+f(n)-\dfrac{1}{k}\dbinom{m+n}{n}$$

$$f(n+1)=\dfrac{kf(n)-\dbinom{m+n}{n}}{k-1}$$

$$f(1)=\dfrac{k^{m+1}-1}{k-1}$$
$O(n\log m)$。
