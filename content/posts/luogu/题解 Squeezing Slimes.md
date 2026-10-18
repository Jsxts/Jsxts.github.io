---
title: "题解 Squeezing Slimes"
date: 2023-09-08T16:37:59+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-squeezing-slimes"
original: "https://www.luogu.com.cn/article/gi15izfl"
math: true
---
提供另一个角度（可能更易于理解）的思路。

首先将操作逆序，即每次将一个区间的数都分成两个数。

我们从左到右贪心地考虑，发现一个 $i$ 最多要 $\lceil\log_2a_i\rceil$ 次操作（记为 $b_i$）。那么是否可以和它左边的操作合并以减少次数呢？

容易发现一个数只有两种可能会优的分解策略：每一次分解时都使得分解出的数升序或降序。如 $7\to 4,3\to2,2,2,1\to1,1,1,1,1,1,1$ 或 $7 \to 3,4\to 1,2,2,2\to1,1,1,1,1,1,1,1$。

那么当 $i$ 往 $i-1$ 合并时，容易发现当 $i-1$ 使用第一种策略时会使可以一起分解的次数减少 $1$。所以我们尽可能使用第二种。

但是当 $b_{i-1}\ge b_i$ 且 $a_i$ 不为 $2$ 的幂时，$i$ 使用第二种就会使一起分解次数减 $1$（会被自己分解出来的 $1$ 隔开）。此时显然用第一种不会劣（就算后面次数少了也不劣）。所以这时我们使用第一种，并将 $lst$（代表上一个可以一起分解的次数）减一。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 1e5;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
int main() {
	int n = read(),lst = 0;
	ll ans = 0;
	for (int i = 1;i <= n;i ++ ) {
		int x = read(),t = ceil(log2(x));
		if ((1 << t) != x && lst >= t) t --;
		ans += max(0,t - lst), lst = t;
	}
	cout << ans;
	return 0;
}
```
