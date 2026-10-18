---
title: "题解：AT_abc345_e [ABC345E] Colorful Subsequence"
date: 2024-03-20T12:56:34+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-at-abc345-e-abc345e-colorful-subsequence"
original: "https://www.luogu.com.cn/article/0s0dumc6"
math: true
---
一个神秘贪心，不太会证但赛时过了。

考虑贪心地删，直到删了 $k$ 个：

- 有相邻相同，找到其中满足和相邻相同并且权值最小的。

- 删掉一个后左右会相同就将其中的最小值一起删掉，并记这类权值最小值为 $w_2$。

- 否则只需删掉一个，记这类权值最小值为 $w_1$。

然后进行判断：

- 若第一类存在，则优先删第一类。

- 否则若只能删 $1$ 个，则删 $w_1$。

- 否则判断：若 $w_2<2w_1$，则删掉那两个，否则删一个。

用链表维护即可。

$O(nk)$。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 2e5;
const int inf = 2e9;
const ll INF = 1e18;
ll read() {
	ll s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-'? -1 : 1, ch = getchar();
	while(isdigit(ch)) s = s * 10 + ch - '0', ch = getchar();
	return s * f;
}
const int mod = 998244353;
int qpow(int a,int b) {
	int res = 1;
	while (b) {
		if (b & 1) res = 1ll * res * a % mod;
		b >>= 1, a = 1ll * a * a % mod;
	}
	return res;
}
int getmod(int x) {
	return x - (x >= mod) * mod;
}
#define x first
#define y second
pair<int,int> a[N + 10];
int pre[N + 10],nxt[N + 10];
void del(int x) {
	pre[nxt[x]] = pre[x];
	nxt[pre[x]] = nxt[x];
}
int main() {
	int T = 1;
	while (T -- ) {
		int n = read(),k = read(),ss = 0;
		ll sum = 0;
		for (int i = 1;i <= n;i ++ ) a[i].x = read(), a[i].y = read(), sum += a[i].y, pre[i] = i - 1, nxt[i] = i + 1, ss += a[i].x == a[i - 1].x;
		if (ss > k) {
			puts("-1");
			continue;
		}
		nxt[0] = 1;
		ll ans = 0;
		while (k -- ) {
			pair<int,int> mn1 = {inf,0},mn2 = {inf,0},mn3 = {inf,0};
			for (int i = 0;i != n + 1;i = nxt[i]) {
				if (!i) continue;
				//cout << "i:" << i << endl;
				if (a[i].x == a[pre[i]].x || a[i].x == a[nxt[i]].x) mn1 = min(mn1,{a[i].y,i});
				else if (a[pre[i]].x == a[nxt[i]].x) mn2 = min(mn2,{a[i].y + min(a[pre[i]].y,a[nxt[i]].y),i});
				else mn3 = min(mn3,{a[i].y,i});
			}
			if (mn1.y != 0) del(mn1.y), ans += mn1.x;//, cout << "!!" << mn1.y << endl;
			else if (mn3.x * 2 < mn2.x || k == 0) del(mn3.y), ans += mn3.x;
			else {
				int p = mn2.y;
//				cout << "!!" << p << endl;
				del(a[pre[p]].y < a[nxt[p]].y ? pre[p] : nxt[p]);
				del(p);
				ans += mn2.x;
				k --;
			}
		}
		
		cout << sum - ans;
	}
	return 0;
}
```
