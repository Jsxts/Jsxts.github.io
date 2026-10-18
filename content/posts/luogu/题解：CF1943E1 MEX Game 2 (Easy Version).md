---
title: "题解：CF1943E1 MEX Game 2 (Easy Version)"
date: 2024-03-22T07:25:42+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-cf1943e1-mex-game-2-easy-version"
original: "https://www.luogu.com.cn/article/ltjkhppf"
math: true
---
首先二分答案。

发现若一个数只剩 $\le k$ 个，那么 Bob 一次就能删光，所以此时 Alice 必须选一个这个数。所以若数量 $\le k$ 的数个数 $>1$ 则 Bob 胜。

另一个观察是 Alice 每次无论如何只会选当前剩余最少的数。

所以，一个显然错误的贪心为：若将 $0\sim mid-1$ 所有数按数量从小到大排序记为 $b$，首先第一个肯定被 Alice 选，之后判断 $b_i-k$ 的前缀和与 $(i-1)k$ 的大小关系即可（即，每次将个数最小的数选到恰剩 $k$ 个，再对下个操作，直到当前最小数已经为 $k$）。

这组 hack 数据可以引出正确思路：

```
3 5
10 9 8 6
```

考虑对 $mid=4$ 的 check，发现会判定为 Alice 胜，但有一个更优的策略：

- Alice 选一个 $3$，序列变为 $10,9,8$。
- Bob 选 $3$ 个 $0$，两个 $1$，序列变为 $7,7,8$。
- Alice 选一个 $0$, 序列变为 $7,8$。
- Bob 选 $2$ 个 $1$，三个 $2$，序列变为 $5,5$，显然此时 Alice 已经输了。

显然 Alice 都是最优决策，那么问题出在之前的错误思路会进行无用的操作，考虑到 Alice 只会选最小的数，不妨令时时刻刻序列中最小的数尽量大，这意味着我们会将损失降到最低，不难证明这是对的。其次，Bob 可以对每个前缀进行上述操作，并取最优结果。因为全部一起不一定最优。

那么，由于 $m$ 较小，考虑直接模拟这个过程，用 deque 即可模拟从大的部分减数以及从最小值删数。

```cpp
#include <bits/stdc++.h>
#define x first
#define y second
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
int m,k,f[N + 10],b[N + 10];
deque<pair<int,int> > q;
pair<int,int> c[N + 10];
bool check(int mid) {
	int ct = 0;
	for (int i = 0;i < mid;i ++ ) ct += f[i] <= k, b[i] = max(0,f[i] - k);
	if (ct >= 2) return 0;
	sort(b,b + mid);
	for (int i = 2;i < mid;i ++ ) {
		int tt = 0;
		for (int j = i;j;j -- ) 
			if (b[j] != b[j + 1] || j == i) c[++tt] = {b[j],1};
			else c[tt].y ++;
		while (q.size()) q.pop_back();
		for (int j = 1;j <= tt;j ++ ) q.push_back(c[j]);
		for (int j = 1;j < i;j ++ ) {
			int tk = k;
			while (1) {
				auto x = q.front();q.pop_front();
				if (!q.size()) {
					if (tk >= 1ll * x.y * x.x) {
						return 0;
					}
					int ct = tk % x.y;
					x.x -= tk / x.y;
					if (ct) q.push_front({x.x - 1,ct});
					q.push_front({x.x,x.y - ct});
					break;
				}
				auto y = q.front();q.pop_front();
				ll val = 1ll * (x.x - y.x) * x.y;
				if (val <= tk) {
					tk -= val, q.push_front({y.x,y.y + x.y});
					if (!y.x) return 0;
					if (!tk) break;
				}
				else {
					int ct = tk % x.y;
					x.x -= tk / x.y;
					if (ct) {
						if (y.x == x.x - 1) q.push_front({y.x,y.y + ct});
						else q.push_front(y), q.push_front({x.x - 1,ct});
					}
					else q.push_front(y);
					q.push_front({x.x,x.y - ct});
					break;
				}
			}
			auto t = q.back();q.pop_back();
			t.y --;
			if (t.y) q.push_back(t);
		}
	}
	return 1;
}
int main() {
	int T = read();
	while (T -- ) {
		m = read(), k = read();
		for (int i = 0;i <= m;i ++ ) f[i] = read();
		int l = 0,r = m + 1;
		while (l < r) {
			int mid = l + r + 1 >> 1;
			if (check(mid)) l = mid;
			else r = mid - 1;
		}
		printf("%d\n",l);
	}
	return 0;
}
```

upd: 如何优化上述做法呢？（参考官方题解）

注意：**由于我们先让 Alice 删去了第一个，接下来每次操作是看成 Bob 先删 $k$ 个，Alice 再删一个。**

考虑这个序列会减到某个时刻，满足 $b_i=\lfloor\dfrac{i+s-1}{n}\rfloor$，其中 $n$ 为序列长度，$s$ 为序列和。在这个状态后，每一次操作后必然还是这个状态，所以可以用二元组 $(n,s)$ 表示。

若我们对这个序列的每个前缀都求出了若干个二元组 $(n,s)$（显然它们可以一起考虑），对于每个 $n$ 最小的 $s$ 最优，所以考虑从 $n\to n-1$：$(n,s)\to (n-1,s-k-\lfloor\dfrac{s-k}{n}\rfloor)$。按 $n$ 从大到小转移完后，我们就求出的对每个 $n$ 最小能转移到的 $s$。只需判断能否转移到 $(1,0)$ 即可。

现在的问题为：对每个前缀求出一开始的时刻满足此时它可以用 $(n,s)$ 表示。这个时刻可以看成：$1\sim p$ 已经被 Alice 选过（这里我们确保操作的任何时刻序列都是递增的，所以这个就是 Alice 的最优策略）。那么 $p$ 要满足的不等式为（令 $pre$ 为 $b$ 的前缀和）：

$$pre_i-pre_p-pk<b_p\times(i-p)$$

发现 $p$ 满足单调性，直接双指针即可。

这一部分：

```cpp
for (int i = 1;i < mid;i ++ ) {
  while (p < i - 1 && pre[i] - pre[p + 1] - 1ll * (p + 1) * k >= 1ll * b[p + 1] * (i - (p + 1))) p ++;
  ans[i - p] = min(ans[i - p],pre[i] - pre[p] - 1ll * p * k);
}
for (int i = mid - 2;i >= 1;i -- ) ans[i] = min(ans[i],ans[i + 1] - k - (ans[i + 1] - k) / (i + 1));
return ans[1] > 0;
```
