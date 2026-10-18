---
title: "题解：AT_arc175_c [ARC175C] Jumping Through Intervals"
date: 2024-03-28T13:53:11+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-at-arc175-c-arc175c-jumping-through-intervals"
original: "https://www.luogu.com.cn/article/vyvbgqja"
math: true
---
困扰我 $3$ 天。

考虑将序列划分为极长的若干段，其中的每一段都满足对于任意 $i$，前缀 $r$ 的最小值 $\ge$ 后缀 $l$ 的最大值。容易证明这样划分后，除了第一段，每一段 $[x,y]$ 都满足 $a_{x-1}<l_x$，这个重要的性质保证了很多决策的正确性。所以第一段需要另外处理。

我们对每段分别处理，记 $l_i$ 的后缀 $\max$ 为 $suf_i$，前缀 $\max$ 为 $pre_i$，并记区间 $r_i$ 最小值为 $mr$。

对于不是第一段的，分情况考虑：

- 若 $mr\ge suf_x$：此时显然每个数都可以取到 $pre_i$。

- 否则，这一段所有数会这样变动：先递增至 $suf_x$（区间 $l_i$ 最大值），再递减至 $mr$，最后一段都是 $mr$。具体地：

	1. 若 $suf_i<mr$，说明之后不会变，必须取到 $mr$。
   2. 否则，由于 $suf_i\ge mr$，说明还会经历上升和下降，现在可以取到 $\min(pre_i,suf_i)$。
   
对于第一段，也可以类似地考虑：

- 若 $mr\ge suf_x$：此时贡献最小的方案是所有数都取 $mr$，因为 $y+1$ 的位置选的数比 $mr$ 大。但是还有一种可能就是 $y=n$，此时应所有数都选 $suf_x$。

- 否则，由于没有上一个数，所以一开始取递增会使答案变劣，所以一开始的那一段从递增改为一直为后缀最大值。

```cpp
#include <bits/stdc++.h>
#define x first
#define y second
using namespace std;
typedef long long ll;
const int inf = 2e9;
const ll INF = 1e15;
const int N = 5e5;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
const int mod = 998244353;
int getmod(int x) {
	return x - (x >= mod) * mod;
}
int qpow(int a,int b) {
	int res = 1;
	while (b) {
		if (b & 1) res = 1ll * res * a % mod;
		b >>= 1, a = 1ll * a * a % mod;
	}
	return res;
}
int n,res[N + 10];
pair<int,int> a[N + 10];
int suf[N + 10];
void solve() {
	for (int i = 1;i <= n;i ++ ) {
		int r1 = i,mr = a[i].y;
		while (r1 < n && a[r1 + 1].x <= mr) mr = min(mr,a[r1 + 1].y), r1 ++;
		suf[r1 + 1] = -inf;
		for (int j = r1;j >= i;j -- ) suf[j] = max(suf[j + 1],a[j].x);
		if (mr >= suf[i]) {
			if (i == 1)
				for (int j = i;j <= r1;j ++ ) res[j] = r1 == n ? suf[i] : mr;
			else {
				int mx = 0;
				for (int j = i;j <= r1;j ++ ) mx = max(mx,a[j].x), res[j] = mx;
			}
		}
		else {
			if (i == 1) {
				for (int j = i;j <= r1;j ++ ) {
					if (suf[j] < mr) res[j] = mr;
					else res[j] = suf[j];
				}
			}
			else {
				int mx = 0;
				for (int j = i;j <= r1;j ++ ) {
					mx = max(mx,a[j].x);
					if (suf[j] < mr) res[j] = mr;
					else res[j] = min(mx,suf[j]);
				}
			}
		}
		i = r1;
	}
}
int main() {
	n = read();
	for (int i = 1;i <= n;i ++ ) a[i].x = read(), a[i].y = read();
	solve();
	for (int i = 1;i <= n;i ++ ) printf("%d ",res[i]);
	return 0;
}
```
