---
title: "题解 [AGC026D] Histogram Coloring"
date: 2023-12-12T10:02:44+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-agc026d-histogram-coloring"
original: "https://www.luogu.com.cn/article/4a9uuj63"
math: true
---
首先容易发现，如果从下往上一行一行填，那么在没有断开前，一行若有两个相邻相同的颜色，则上一行的染色法方案唯一确定，否则上一行有两种填法，都满足和这一行一样也是红蓝交替。

接下来考虑 dp，设 $f_{l,r,0/1,d}$ 表示区间 $[l,r]$ 已经填了从下往上的 $d$ 行，并且钦定当前是/不是有唯一填法（是/不是红蓝交替）的方案数。

那么，首先找到区间的最小值位置 $p$，那么填到这个高度后会分成两端区间。

是红蓝交替的较容易转移：

$$f_{l,r,0,d}=f_{l,p-1,0,h_p}\times f_{p + 1,r,0,h_p}\times 2^{h_p-d}$$

因为每行都有 $2$ 种填法。

不是红蓝交替，那么从 $d$ 填到 $h_p$ 是唯一确定的，只需要对第 $h_p$ 行的状况分类讨论（称 $[l,p-1]$ 为左侧，$[p+1,r]$ 为右侧）：

首先若一侧在断开后仍为不交替，那么下方的填法会在更上面的决策中完成，所以不需要计算方案。若为交替，则对应的下方填法有两种：以红或蓝开始交替。同时 $p$ 位置填什么数还没有确定。

- 左侧交替，右侧交替：这时一定满足 $p$ 的位置要与 $p-1$ 或 $p+1$ 相同（否则违反不是红蓝交替的条件），枚举两边下方的填法共有 $6$ 种。

- 剩下的情况 $p$ 位置都没有限制，填任何数都行。

所以转移如下：

$$\begin{aligned}
f_{l,r,1,d}=6\times f_{l,p-1,0,d} \times f_{p+1,r,0,d}\\
+4\times f_{l,p-1,0,d} \times f_{p+1,r,1,d}\\
+4\times f_{l,p-1,1,d} \times f_{p+1,r,0,d}\\
+2\times f_{l,p-1,1,d} \times f_{p+1,r,1,d}
\end{aligned}$$

可能要处理一下 $p=l,r$ 的情况。

虽然数组开不下，但是状态数显然是 $O(n^3)$ 的，直接记搜即可。

代码：
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 1e2;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
int n,h[N + 10],mn[N + 10][N + 10];
const int mod = 1e9 + 7,inv2 = mod + 1 >> 1;
map<ll,int> mp;
ll hsh(int l,int r,int f,int d) {
	return 1ll * d * n * n * 2 + 1ll * l * n * 2 + 1ll * r * 2 + f;
}
int qpow(int a,int b) {
	int res = 1;
	while (b) {
		if (b & 1) res = 1ll * res * a % mod;
		b >>= 1, a = 1ll * a * a % mod;
	}
	return res;
}
int dfs(int l,int r,int f,int d) {//f:当前为/不为01交替 
	if (l > r) return 1;
	if (l == r) return f ? 0 : qpow(2,h[l] - d);
	ll hs = hsh(l,r,f,d);
	if (mp.count(hs)) return mp[hs];
	int p = mn[l][r];
	if (!f) return mp[hs] = 1ll * dfs(l,p - 1,f,h[p]) * dfs(p + 1,r,f,h[p]) % mod * qpow(2,h[p] - d) % mod;
	if (p == l) return mp[hs] = 2ll * (dfs(p + 1,r,0,h[p]) + dfs(p + 1,r,1,h[p])) % mod;
	if (p == r) return mp[hs] = 2ll * (dfs(l,p - 1,0,h[p]) + dfs(l,p - 1,1,h[p])) % mod;
	int t00 = 6ll * dfs(l,p - 1,0,h[p]) % mod * dfs(p + 1,r,0,h[p]) % mod;
	int t10 = 4ll * dfs(l,p - 1,1,h[p]) % mod * dfs(p + 1,r,0,h[p]) % mod;
	int t01 = 4ll * dfs(l,p - 1,0,h[p]) % mod * dfs(p + 1,r,1,h[p]) % mod;
	int t11 = 2ll * dfs(l,p - 1,1,h[p]) % mod * dfs(p + 1,r,1,h[p]) % mod;
	return mp[hs] = ((ll)t00 + t10 + t01 + t11) % mod;
}
int main() {
	n = read();
	for (int i = 1;i <= n;i ++ ) h[i] = read();
	for (int i = 1;i <= n;i ++ ) {
		mn[i][i] = i;
		for (int j = i + 1;j <= n;j ++ )
			mn[i][j] = h[j] < h[mn[i][j - 1]] ? j : mn[i][j - 1];
	}
	cout << (dfs(1,n,1,0) + dfs(1,n,0,0)) % mod;
	return 0;
}
```
