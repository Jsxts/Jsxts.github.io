---
title: "斜率优化 DP"
date: 2023-09-27T07:45:50+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "斜率优化-dp"
original: "https://www.luogu.com.cn/article/zsvf92zz"
math: true
---
基本形式： $dp_i=\min\{dp_j+A(i)B(j)+C(i)+D(j)+E\}$（$\max$ 类似）

将 $\min$ 去掉，提出常数和只与 $i$ 有关的项：

$$dp_i-C(i)-E=dp_j+A(i)B(j)+D(j)$$

之后尝试比较两个决策点谁更优，若 $j_1$ 比 $j_2$ 更优（这里设 $B(j_1)>B(j_2)$）：

$$dp_{j_1}+A(i)B(j_1)+D(j_1)<dp_{j_2}+A(i)B(j_2)+D(j_2)$$

$$A(i)<\dfrac{(dp_{j_2}+D(j_2))-(dp_{j_1}+D(j_1))}{B(j_1)-B(j_2)}$$

那么令 $X(j)=-B(j),Y(j)=dp_j+D(j)$，则：

$$A(i)<\dfrac{Y(j_2)-Y(j_1)}{X(j_2)-X(j_1)}$$

这个式子和斜率的计算式类似，于是我们考虑将所有的决策点放到二维平面上。那么由 $X(j)=-B(j),B(j_1)>B(j_2)$ 得出，当两点斜率 $>A(i)$ 时，横坐标小的更优，反之横坐标大的更优。

然后由推出的式子，我们考虑三个点 $A,B,C$ 之间的关系：

如图，此时 $k_{BC}<k_{AB}$，那么：

- 若 $A(i)<k_{BC}$，那么 $B$ 优于 $C$，$A$ 优于 $B$。
- 若 $k_{BC}\le A(i)<k_{AB}$，那么 $C$ 优于 $B$，$A$ 优于 $B$。
- 若 $k_{AB}\le A(i)$，那么 $C$ 优于 $B$，$B$ 优于 $A$。

此时我们发现 $B$ 永远不会是最优决策，可以在二维平面中直接将 $B$ 去掉。

所以，所有可能成为决策点的点形成了一个下凸壳，并且相邻点斜率递增。我们用单调栈将凸壳维护出来。（当然这个凸包的方向取决于推出的不等式是什么。）

那么我们对于一个 $A(i)$ 查询最优决策可以看成在凸壳中从后往前遍历，斜率 $>A(i)$ 代表更优的点还在前面，直到找到第一个点满足和上一个点的斜率 $\le A(i)$ 则答案就是这个点。（这个操作实质上就是找到斜率为 $A(i)$ 的直线在这个凸壳上的切点。）又因为斜率递增，可以二分求出。

但是，大多数题目都有更好的性质：$A(i)$ 递增/递减，也就是我们无需在凸包上二分，用单调队列弹出头部斜率 $\le A(i)$ 的点即可。

细节问题：

- 凸包中出现三点共线可以直接去掉中间的点。

- 出现 $B(j_1)=B(j_2)$ 时，由式子知若 $Y(j_1)<Y(j_2)$ 则 $j_1$ 优于 $j_2$，所以将横坐标相同的斜率设成 $\infty$ 即可。

- 计算斜率时可以将除法转为乘法来避免精度误差。

例题：P6302 [NOI2019] 回家路线

不是最模板的例题。

DP 主体看起来有 $3$ 个：站点、火车、时刻。尝试后发现用火车作为 DP 主体是最优的。

不管最后增加的，设 $dp_i$ 为坐完第 $i$ 辆列车后的最小烦躁值。容易写出转移：

$$dp_i=\min\{dp_j+A(p_i-q_j)^2+B(p_i-q_j)+C\}$$

其中 $j$ 要满足 $y_j=x_i,q_j\le p_i$。

先不管 $j$ 需要满足的条件，将转移式写成斜率优化的形式：

令 $A(i)=2Ap_i,B(j)=-q_j,C(i)=Ap_i^2+Bp_i,D(j)=Aq_j^2-Bq_j,E=C$，则:

$$dp_i=\min\{dp_j+A(i)B(j)+C(i)+D(j)+E\}$$

分析过程与上面类似，最后设 $X(j)=-B(j)=q_j,Y(j)=dp_j+D(j)=dp_j+Aq_j^2-Bq_j$。这样我们就可以得到一个下凸壳。若我们按 $p_i$ 从小到大转移则满足 $A(i)$ 递增，可以用单调队列维护凸壳。

对于剩下的两个对 $j$ 的限制：

- $y_j=x_i$，开 $n$ 个单调队列即可。

- $q_j\le p_i$，也就是只能用 $x$ 坐标 $\le p_i$ 的点转移，又因为 $p_i$ 递增，于是每次 $p_i$ 增大时将新的满足条件的决策点加入即可。又因为 $X(j)=q_j$ 在加入时也是递增的所以可以正确维护凸包。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e18;
const int N = 1e5;
const int M = 1e6;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
int n,m,A,B,C;
struct node {
	int x,y,p,q;
}a[M + 10];
int cmp(node x,node y) {
	return x.p == y.p ? x.q < y.q : x.p < y.p;
}
ll dp[M + 10];
inline int X(int j) {return a[j].q;}
inline ll Y(int j) {return dp[j] + 1ll * A * a[j].q * a[j].q - 1ll * B * a[j].q;}
vector<int> qu[N + 10];
vector<int> vc[N + 10];
double slope(int x,int y) {
	return X(x) == X(y) ? -INF : (Y(y) - Y(x)) * 1.0 / (X(y) - X(x));
}
bool comp(int x,int y,int z) {//比较 kxy 和 kyz 的大小 
	if (X(x) == X(y)) return 1;
	if (X(y) == X(z)) return 0;
	//(Y(y)-Y(x))/(X(y)-X(x))>(Y(z)-Y(y))/(X(z)-X(y))
	//(Y(y)-Y(x))*(X(z)-X(y))>(Y(z)-Y(y))*(X(y)-X(x))
	return (Y(y) - Y(x)) * (X(z) - X(y)) >= (Y(z) - Y(y)) * (X(y) - X(x));
}
int main() {
	n = read(), m = read(), A = read(), B = read(), C = read();
	for (int i = 1;i <= m;i ++ ) a[i].x = read(), a[i].y = read(), a[i].p = read(), a[i].q = read();
	sort(a + 1,a + m + 1,cmp);
	for (int i = 1;i <= m;i ++ ) vc[a[i].q].push_back(i);
	int r = 0;
	qu[1].push_back(0);
	for (int i = 1;i <= m;i ++ ) {
		int x = a[i].x,y = a[i].y,p = a[i].p,q = a[i].q;
		for (;r <= p;r ++ )
			for (auto v : vc[r]) {
				int ty = a[v].y;
				if (dp[v] == INF) continue;
				while (qu[ty].size() > 1 && comp(qu[ty][qu[ty].size()-2],qu[ty].back(),v)) qu[ty].pop_back();
				qu[ty].push_back(v);
			}
		while (qu[x].size() > 1 && slope(qu[x][0],qu[x][1]) < 2ll * A * p) qu[x].erase(qu[x].begin());
		if (qu[x].size()) {
			int h = qu[x].front();
			dp[i] = dp[h] + 1ll * A * (p - a[h].q) * (p - a[h].q) + 1ll * B * (p - a[h].q) + C;
		}
		else dp[i] = INF;
	}
	ll ans = INF;
	for (int i = 1;i <= m;i ++ ) if (a[i].y == n) ans = min(ans,dp[i] + a[i].q);
	cout << ans;
	return 0;
}
```
