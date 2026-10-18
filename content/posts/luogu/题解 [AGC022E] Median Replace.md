---
title: "题解 [AGC022E] Median Replace"
date: 2023-12-04T20:11:59+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-agc022e-median-replace"
original: "https://www.luogu.com.cn/article/jqosexwv"
math: true
---
不同的做法！

仍然是先考虑 check，一开始容易发现 $1$ 的个数 $>0$ 的个数肯定有解，但这还远远不够。

又发现若最左边和最右边都为 $1$，则肯定可以将中间的 $0$ 数量减成至多 $1$ 个后满足条件。受这个启发，我们期望寻找到两个 $1$（设为 $l,r$），满足可以消去它们左边和右边的 $0$ 后只剩下这个区间的数（当然旁边有更多的 $1$ 也无妨）。发现只要求出最小的 $l$ 和最大 $r$ 满足 $l\le r$ 即可合法。现在尝试求最小 $l$（最大 $r$ 同理）。

发现我们有两个操作，一个是消去一对 $01$，一个是将 $0$ 的数量减 $2$。

所以我们考虑从左到右扫，记录当前最少 $0$ 的数量 $c$，加入一个 $1$ 就将 $c$ 减一，加入 $0$ 就加一后若 $c=3$ 就变为 $c=1$（贪心地删去 $0$ 连续段）。

当一个 $1$ 左侧前缀的 $c=0$ 时，证明这个 $1$ 作为 $l$ 是合法的。

像这样 check，经过暴力验证是正确的。

现在考虑 dp。设 $dp_{i,j}$ 表示前 $i$ 个数，$c=j$ 的方案数。转移就讨论这一个填 $0$ 或 $1$ 然后再简单地转移，发现第二维只会取到 $0\sim 2$，所以直接 dp 即可。

在 dp 的同时记录 $L_i$ 表示 $i$ 作为上文的 $l$ 的方案数。统计就在第 $i$ 位取到 $1$ 时，令 $L_i\leftarrow L_i+dp_{i-1,0}$。

同理，倒着 dp 可以得到 $R_i$ 表示 $i$ 作为 $r$ 的方案数。

计算方案数就是统计如下式子：

$$\sum_{i=1}^n\sum_{j=i}^nL_iR_j2^{p_{j-1}-p_i}$$

其中 $p_i$ 为前缀问号个数。

这个可以轻松优化，注意 $i=j$ 要单独计算。

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 3e5;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
char s[N + 10];
int dp[N + 10][3],L[N + 10],R[N + 10];
const int mod = 1e9 + 7,inv2 = mod + 1 >> 1;
int getmod(int x) {
	return x - (x >= mod) * mod;
}
int pw[N + 10],pre[N + 10],ipw[N + 10];
int main() {
	scanf("%s",s + 1);
	int n = strlen(s + 1);pw[0] = ipw[0] = 1;
	for (int i = 1;i <= n;i ++ )
		pw[i] = 2ll * pw[i - 1] % mod, pre[i] = pre[i - 1] + (s[i] == '?'), ipw[i] = 1ll * ipw[i - 1] * inv2 % mod;
	dp[0][0] = 1;
	for (int i = 0;i < n;i ++ ) {
		for (int j = 0;j < 2;j ++ ) {
			if (s[i + 1] != j + '0' && s[i + 1] != '?') continue;
			if (j == 1) {
				L[i + 1] = getmod(L[i + 1] + dp[i][0]);
				for (int k = 0;k < 2;k ++ ) {
					dp[i + 1][k] = getmod(dp[i + 1][k] + dp[i][k + 1]);
				}
			}
			else {
				for (int k = 1;k < 3;k ++ )
					dp[i + 1][k] = getmod(dp[i + 1][k] + dp[i][k - 1]);
				dp[i + 1][1] = getmod(dp[i + 1][1] + dp[i][2]);
			}
		}
	}
	memset(dp,0,sizeof dp);
	dp[n + 1][0] = 1;
	for (int i = n + 1;i >= 2;i -- ) {
		for (int j = 0;j < 2;j ++ ) {
			if (s[i - 1] != j + '0' && s[i - 1] != '?') continue;
			if (j == 1) {
				R[i - 1] = getmod(R[i - 1] + dp[i][0]);
				for (int k = 0;k < 2;k ++ ) {
					dp[i - 1][k] = getmod(dp[i - 1][k] + dp[i][k + 1]);
				}
			}
			else {
				for (int k = 1;k < 3;k ++ )
					dp[i - 1][k] = getmod(dp[i - 1][k] + dp[i][k - 1]);
				dp[i - 1][1] = getmod(dp[i - 1][1] + dp[i][2]);
			}
		}
	}
	int ans = 0;
	for (int i = 1;i <= n;i ++ ) ans = (ans + 1ll * L[i] * R[i]) % mod;
	for (int i = 1;i <= n;i ++ ) L[i] = 1ll * L[i] * ipw[pre[i]] % mod, R[i] = 1ll * R[i] * pw[pre[i - 1]] % mod;
	for (int i = n;i;i -- ) R[i] = getmod(R[i] + R[i + 1]);
	for (int i = 1;i <= n;i ++ ) ans = (ans + 1ll * L[i] * R[i + 1]) % mod;
	cout << ans;
	return 0;
}
/*
?0101
*/
```
