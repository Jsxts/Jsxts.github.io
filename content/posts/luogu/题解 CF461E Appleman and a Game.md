---
title: "题解 CF461E Appleman and a Game"
date: 2023-12-07T09:06:06+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-cf461e-appleman-and-a-game"
original: "https://www.luogu.com.cn/article/q6fv4g3g"
math: true
---
考虑知道了 $s$ 怎么求它的最小操作次数：设 $dp_i$ 表示前 $i$ 个的最小操作次数，然后找到以 $i$ 结尾在 $t$ 中出现过的最长串，设其长度为 $l$，从 $dp_{i-l-1}+1$ 转移过来。由于 dp 值随 $i$ 不降，所以只需找到最长串。

现在考虑使得每次这个 dp 的转移时的 $l$ 都尽量短（这样会使上面的 dp 值尽量大）。假设当前结尾处字母为 $c$，那么我们需要找到一个以 $c$ 结尾并在 $t$ 中没有出现的最短的串 $s$，贪心地填在这里，那么划分时考虑从后往前划分，肯定是将 $s$ 除去第一个字母的剩下部分划成一段，剩下再同样的继续贪心（举例，若以 $\texttt{A}$ 结尾时 $s=\texttt{ADA}$，则肯定是将 $\texttt{DA}$ 划做一段，$\texttt{A}$ 与剩下的串再继续划分）。

所以，这个串只和长度、开头、结尾有关。于是我们可以预处理出 $val_{i,j}$ 表示开头为 $i$，结尾为 $j$ 且没在 $t$ 中出现的最短串长度。之后考虑一个 dp：设 $f_{i,j}$ 表示当 $s$ 长度为 $i$，结尾为 $j$ 时的最小操作次数的最大值。转移就枚举开头的字母 $k$，从 $f_{i-val_{k,j}+1,k}+1$ 转移过来。

但是有可能最后一段不是由一个最短串组成，那么考虑在开头转移时可以只填最短串的一段前缀（即 $i<val_{k,j}$ 时将 $f_{i,j}$ 设为 $1$），这样就钦定最后答案一定是 $\max\{f_{n,0\sim 3}\}$。

然后发现这个 $val_{i,j}$ 长度是 $\log_4(|t|)$ 级别的，所以可以存下 $i$ 前 $\log_4(|t|)$ 个下标的所有状态共 $|\sum|\log_4(|t|)$ 个状态进行矩阵加速即可。

而对于前面的 $val_{i,j}$ 如何预处理，同样因为长度很短所以直接枚举长度然后判断是否每种这个长度的串都在 $t$ 中出现过。

总时间复杂度 $O((|\sum|\log_4(|t|))^3+|t|\log_4(|t|))$。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 1e5;
inline ll read() {
	ll s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
int f[N + 10][5],val[4][4],l;
ll n;
string s;
struct Matrix {
	ll w[45][45];
	ll* operator[] (const int x) {
		return w[x];
	}
	friend Matrix operator * (Matrix x,Matrix y) {
		Matrix z;
		memset(z.w,-0x3f,sizeof z.w);
		for (int i = 0;i < 40;i ++ )
			for (int k = 0;k < 40;k ++ )
				for (int j = 0;j < 40;j ++ )
					z[i][j] = max(z[i][j],x[i][k] + y[k][j]);
		return z;
	}
}A;
int id(int i,int j) {
	return (i - 1) * 4 + j;
}
Matrix qpow(Matrix a,ll b) {
	Matrix res;
	memset(res.w,0,sizeof res.w);
	while (b) {
		if (b & 1) res = res * a;
		b >>= 1, a = a * a;
	}
	return res;
}
map<string,int> mp;
int main() {
	n = read();
	cin >> s;
	l = s.size();s = '#' + s;
	for (int i = 2;i <= 10;i ++ ) {
		for (char a = 'A';a <= 'D';a ++ )
			for (char b = 'A';b <= 'D';b ++ ) {
				if (val[a - 'A'][b - 'A']) continue;
				int ct = 0;
				mp.clear();
				for (int j = 1;j <= l - i + 1;j ++ )
					if (s[j] == a && s[j + i - 1] == b) {
						string ss = s.substr(j,i);
						if (!mp[ss]) mp[ss] = 1, ct ++;
					}
				if (ct < pow(4,i - 2)) val[a - 'A'][b - 'A'] = i;
			}
	}
	memset(A.w,-0x3f,sizeof A.w);
	for (int i = 0;i < 36;i ++ ) A[i + 4][i] = 0;
	for (int j = 0;j < 4;j ++ )
		for (int k = 0;k < 4;k ++ ) {
			A[id(10 - val[k][j] + 2,k)][id(10,j)] = 1;
		}
	Matrix ans = qpow(A,n);
	ll Ans = max({ans[0][id(10,0)],ans[0][id(10,1)],ans[0][id(10,2)],ans[0][id(10,3)]});
	cout << Ans;
	return 0;
}
```
