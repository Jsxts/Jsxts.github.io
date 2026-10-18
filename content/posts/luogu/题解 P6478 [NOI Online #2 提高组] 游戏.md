---
title: "题解 P6478 [NOI Online #2 提高组] 游戏"
date: 2023-03-25T16:34:04+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-p6478-noi-online-2-提高组-游戏"
original: "https://www.luogu.com.cn/article/kg1xetto"
math: true
---
看到题目说的**恰好 $k$ 个**,我们可以想到二项式反演：

设 $f_n$ 为恰好选 $n$ 个有标号元素满足某些条件的方案数， $g_n$ 表示至少选 $n$ 个的方案数，则 $f_n=\sum_{i=n}^m\dbinom{i}{n}(-1)^{i-n}g_i$。证明的话，就假设这个式子成立，再用 $f_i$ 替换 $g_i$，推完式子最后会发现 $f_n=f_n$，假设成立。

有了这个有什么用呢？我们就可以转而去算 $g_n$，这可以在树中进行树形 DP 求出。具体的，设 $dp_{x,i}$ 表示 $x$ 子树内，钦定有 $i$ 组分出胜负的方案数（只考虑这 $i$ 组）。转移类似树形背包，最后再加上钦定选 $x$ 这个点造成的贡献即可。那么 $g_i=dp_{1,i}\times (m-i)!$（没钦定的可以随便选）。之后再用 $g$ 算出 $f$ 即可。
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 2e9;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
//dp[i][j]:i子树内，钦定有j组分出胜负的方案数（只考虑这j组） 
int n,dp[5010][5010],tmp[5010];
char s[5010];
const int mod = 998244353;
struct edge {
	int v,nxt;
}ed[10010];
int head[5010],cnt,a[5010],sz[5010];
void add(int u,int v) {
	ed[++cnt] = {v,head[u]};
	head[u] = cnt;
}
void dfs(int x,int fa) {
	dp[x][0] = 1;
	sz[x] = 1, a[x] = (s[x] == '0');
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == fa) continue;
		dfs(v,x);
		for (int j = 0;j <= sz[x];j ++ ) tmp[j] = dp[x][j], dp[x][j] = 0;
		for (int j = 0;j <= sz[x];j ++ )
			for (int k = 0;k <= sz[v];k ++ )
				dp[x][j + k] = (dp[x][j + k] + 1ll * tmp[j] * dp[v][k]) % mod;
		sz[x] += sz[v];
		a[x] += a[v];
	}
	for (int i = min(sz[x] - a[x],a[x]);i;i -- ) {
		if (s[x] == '0') dp[x][i] = (dp[x][i] + 1ll * dp[x][i - 1] * (sz[x] - a[x] - (i - 1))) % mod;
		else dp[x][i] = (dp[x][i] + 1ll * dp[x][i - 1] * (a[x] - (i - 1))) % mod;
	}
}
int qpow(int a,int b) {
	int res = 1;
	while (b) {
		if (b & 1) res = 1ll * res * a % mod;
		b >>= 1, a = 1ll * a * a % mod;
	}
	return res;
}
int g[5010],f[5010],jc[5010],ivjc[5010];
int C(int n,int m) {
	return 1ll * jc[n] * ivjc[m] % mod * ivjc[n - m] % mod;
}
int main() {
	n = read();
	scanf("%s",s+1);
	for (int i = 1;i < n;i ++ ) {
		int u = read(),v = read();
		add(u,v), add(v,u);
	}
	dfs(1,0);jc[0] = ivjc[0] = 1;
	for (int i = 1;i <= n;i ++ ) jc[i] = 1ll * jc[i - 1] * i % mod;
	ivjc[n] = qpow(jc[n],mod - 2);
	for (int i = n - 1;i;i -- ) ivjc[i] = 1ll * ivjc[i + 1] * (i + 1) % mod;
	for (int i = 0;i <= n / 2;i ++ ) g[i] = 1ll * dp[1][i] * jc[n / 2 - i] % mod;
	for (int i = 0;i <= n / 2;i ++ ) {
		int res = 0;
		for (int j = i;j <= n / 2;j ++ ) res = ((res + 1ll * C(j,i) * g[j] * (j - i & 1 ? -1 : 1)) % mod + mod) % mod;
		printf("%d\n",res);
	}
	return 0;
}

```
