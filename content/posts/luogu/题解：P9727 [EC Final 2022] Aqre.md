---
title: "题解：P9727 [EC Final 2022] Aqre"
date: 2024-03-22T11:02:14+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-p9727-ec-final-2022-aqre"
original: "https://www.luogu.com.cn/article/yj6ypex8"
math: true
---
首先特判 $n,m\le 3$。

其次是其中一个 $\le 3$（假设为 $n$）：

- $n=2$：形如

```
11101110
10111011
```

- $n=3$：形如

```
1110111011
1011101110
1110111011
```

接下来是一般情况。我们猜测（？答案一定在每个 $4\times 4$ 矩形里同构，并满足矩形内为 $4$ 个不同行、列的点。

一种比较暴力的手段为枚举 $4\times 4$ 矩形的所有方案，并 check 每种是否合法（连通）。

不过通过若干手玩，可以发现答案的构成会有以下形式：

```
0111
1101
1011
1110

1110
1011
1101
0111

1101
1110
1011
0111

1110
1101
0111
1011
```

将所有答案排个序即可。注意其中第 $3,4$ 种分别在 $m\equiv0\pmod 4,n\equiv0\pmod 4$ 时不合法。

```cpp
#include <bits/stdc++.h>
#define x first
#define y second
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 1e3;
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
int id[10];
struct node {
	int ct;
	int a[N + 10][N + 10];
}a[6];
int cmp(int x,int y) {
	return a[x].ct < a[y].ct;
}
int main() {
	int T = read();
	while (T -- ) {
		int n = read(),m = read();
		a[0].ct = n * m;
		a[1].ct = a[2].ct = a[3].ct = a[4].ct = a[5].ct = 0;
		for (int i = 1;i <= n;i ++ )
			for (int j = 1;j <= m;j ++ ) a[0].a[i][j] = a[1].a[i][j] = a[2].a[i][j] = a[3].a[i][j] = a[4].a[i][j] = a[5].a[i][j] = 1;
		if (n <= 3 && m > 3) {
			for (int i = 4;i <= m;i += 4) a[0].a[1][i] = a[0].a[3][i] = 0, a[0].ct --, a[0].ct -= n == 3;
			for (int i = 2;i <= m;i += 4) a[0].a[2][i] = 0, a[0].ct --;
		}
		else if (m <= 3 && n > 3) {
			for (int i = 4;i <= n;i += 4) a[0].a[i][1] = a[0].a[i][3] = 0, a[0].ct --, a[0].ct -= m == 3;
			for (int i = 2;i <= n;i += 4) a[0].a[i][2] = 0, a[0].ct --;
		}
		else {
			a[3].ct = a[4].ct = a[5].ct = n * m;
		}
		if (n > 3 && m > 3) {
			for (int i = 1;i <= n;i += 4)
				for (int j = 4;j <= m;j += 4)
					a[0].a[i][j] = 0, a[0].ct --;
			for (int i = 2;i <= n;i += 4)
				for (int j = 2;j <= m;j += 4)
					a[0].a[i][j] = 0, a[0].ct --;
			for (int i = 3;i <= n;i += 4)
				for (int j = 3;j <= m;j += 4)
					a[0].a[i][j] = 0, a[0].ct --;
			for (int i = 4;i <= n;i += 4)
				for (int j = 1;j <= m;j += 4)
					a[0].a[i][j] = 0, a[0].ct --;

			for (int i = 1;i <= n;i += 4)
				for (int j = 1;j <= m;j += 4)
					a[3].a[i][j] = 0, a[3].ct --;
			for (int i = 2;i <= n;i += 4)
				for (int j = 3;j <= m;j += 4)
					a[3].a[i][j] = 0, a[3].ct --;
			for (int i = 3;i <= n;i += 4)
				for (int j = 2;j <= m;j += 4)
					a[3].a[i][j] = 0, a[3].ct --;
			for (int i = 4;i <= n;i += 4)
				for (int j = 4;j <= m;j += 4)
					a[3].a[i][j] = 0, a[3].ct --;

			if (m % 4 != 0) {
				a[1].ct = n * m;
				for (int i = 1;i <= n;i += 4)
					for (int j = 3;j <= m;j += 4)
						a[1].a[i][j] = 0, a[1].ct --;
				for (int i = 2;i <= n;i += 4)
					for (int j = 4;j <= m;j += 4)
						a[1].a[i][j] = 0, a[1].ct --;
				for (int i = 3;i <= n;i += 4)
					for (int j = 2;j <= m;j += 4)
						a[1].a[i][j] = 0, a[1].ct --;
				for (int i = 4;i <= n;i += 4)
					for (int j = 1;j <= m;j += 4)
						a[1].a[i][j] = 0, a[1].ct --;
			}
			if (n % 4 != 0) {
				a[2].ct = n * m;
				for (int i = 1;i <= n;i += 4)
					for (int j = 4;j <= m;j += 4)
						a[2].a[i][j] = 0, a[2].ct --;
				for (int i = 2;i <= n;i += 4)
					for (int j = 3;j <= m;j += 4)
						a[2].a[i][j] = 0, a[2].ct --;
				for (int i = 3;i <= n;i += 4)
					for (int j = 1;j <= m;j += 4)
						a[2].a[i][j] = 0, a[2].ct --;
				for (int i = 4;i <= n;i += 4)
					for (int j = 2;j <= m;j += 4)
						a[2].a[i][j] = 0, a[2].ct --;
			}
		}
		for (int i = 0;i < 4;i ++ ) id[i] = i;
		sort(id,id+4,cmp);
		printf("%d\n",a[id[3]].ct);
		for (int i = 1;i <= n;i ++ ,puts(""))
			for (int j = 1;j <= m;j ++ )
				putchar(a[id[3]].a[i][j] + '0');
	}
	return 0;
}
```
