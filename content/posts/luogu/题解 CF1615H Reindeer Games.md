---
title: "题解 CF1615H Reindeer Games"
date: 2023-10-25T11:37:35+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-cf1615h-reindeer-games"
original: "https://www.luogu.com.cn/article/3itukhpp"
math: true
---
看到数据范围比较小，考虑转化成线性规划标准型后用单纯形求解。

那么将对每个数加多少和减多少拆成两个变量，限制转换为：

$$a_i+x_i-x'_i\le a_j+x_j-x'_{j}$$
$$x_i-x'_i-x_j+x'_j\le a_j- a_i$$

套用单纯形模板求即可。直接做会 TLE，在 pivot 函数加一个优化就可以过了。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 1e3;
const double eps = 1e-6;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}

ll ans[N * 4 + 10];
struct Simplex {
	double A[N * 2 + 10][N * 2 + 10];
	int id[N * 4 + 10],n,m;
	void pivot(int r,int c) {
		swap(id[r + n],id[c]);
		double t = A[r][c];
		A[r][c] = 1;
		for (int i = 0;i <= n;i ++ ) A[r][i] /= t; 
		vector<int> vc;
		for (int i = 0;i <= n;i ++ ) if (fabs(A[r][i]) > eps) vc.push_back(i);
		for (int i = 0;i <= m;i ++ ) {
			if (abs(A[i][c]) > eps && i != r) {
				t = A[i][c], A[i][c] = 0;
				for (int j : vc) A[i][j] -= t * A[r][j];
			}
		}
	}
	void init() {
		for (int i = 1;i <= n;i ++ ) id[i] = i;
		while (1) {
			int r = 0,c = 0;
			for (int i = 1;i <= m;i ++ )
				if (A[i][c] < -eps && (!r || rand() & 1)) r = i;
			if (!r) return;
			for (int i = 1;i <= n;i ++ )
				if (A[r][i] < -eps && (!c || rand() & 1)) c = i;
			if (!c) {
				puts("-1");
				exit(0);
			}
			pivot(r,c);
		}
	}
	void simplex() {
		while (1) {
			int r = 0,c = 0;
			double mx = eps;
			for (int i = 1;i <= n;i ++ )
				if (A[0][i] > mx) c = i, mx = A[0][i];
			if (!c) return;
			double mn = INF;
			for (int i = 1;i <= m;i ++ )
				if (A[i][c] > eps && A[i][0] / A[i][c] < mn)
					r = i, mn = A[i][0] / A[i][c];
			if (!r) puts("Unbounded"), exit(0);
			pivot(r,c);
		}
	}
	void get_p() {
		for (int i = 1;i <= m;i ++ ) ans[id[i + n]] = ll(A[i][0] + 0.5);
	}
	// n 变量数 m 限制数 
	//A[0][i]:标准形中的 c[i]
	//A[i][j]:标准形中的 a[i][j]
	//A[i][0]:原为标准形中的 b[j]，经过算法后变为标准形中的 x[j]（即解） 
	//-A[0][0]:max z
	//注意此代码处理的限制为Σa[i][j]*x[j]<=b[i] 而非 = 
	//调用init()初始化，若所有 x[i] = 0 是一组可行解就不用初始化
	//调用simplex()进行算法 
	//调用get_project()得出方案 
	//注意数组开多少要自己调下 
}S;
int a[N + 10];
int main() {
	int n = read(),m;S.n = n * 2;
	m = S.m = read();
	for (int i = 1;i <= n;i ++ ) a[i] = read();
	for (int i = 1;i <= 2 * n;i ++ ) S.A[0][i] = -1;
	for (int i = 1;i <= m;i ++ ) {
		int x = read(),y = read();
		S.A[i][x + n] = -1, S.A[i][x] = 1;
		S.A[i][y + n] = 1, S.A[i][y] = -1;
		S.A[i][0] = a[y] - a[x];
	}
	S.init(), S.simplex(), S.get_p();
	for (int i = 1;i <= n;i ++ ) printf("%lld ",ans[i] - ans[i + n] + a[i]);
	return 0;
}
```
