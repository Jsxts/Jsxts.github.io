---
title: "题解 Lazy Cow"
date: 2024-02-21T17:08:31+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-lazy-cow"
original: "https://www.luogu.com.cn/article/i5wlq9ym"
math: true
---
贪心+模拟。

首先这个序列任何时候都递减。

加入一条限制，首先判断是否本来已经满足条件。之后，容易发现理想情况肯定是恰好满足限制，并将 $1\sim x$ 中的数尽量均分。我们称这样的分配方案为“变成这段和的平均分”。

那么由于之前可能有 $x'<x$ 的其它限制使得不能用上面的方案，并且仍然是尽量平均分最优，我们容易得到结论：我们会将一段以 $x$ 结尾的极长后缀修改为这一段和的平均分，并且不修改剩下的部分使得所有限制仍然合法并且 $1\sim x$ 的和恰好为 $y$。

那么做出这个修改还会影响 $x'>x$ 的其他限制（因为前缀和变大所以可以减小一些位置的数）。考虑将之前的所有限制的位置存下来（它们会将这个序列分成若干段）。由于还是要时刻满足序列递减，所以不难想到是往之后的每一段考虑：使这一段整体减若干次直到再减就不能使序列递减。这样之后，这一段就可以和往后的一段合并（也就是这个限制没有用了！）。当然，可以将数减小的前提肯定是前缀和不会比之前要小。

举个例子：
```
3
2 10
6 20
3 15
```

第一个限制使序列变为 `5 5`，第二个限制出现时，若没有第一个限制肯定是 `4 4 3 3 3 3` 最优，由于第一个限制只能变为 `5 5 3 3 2 2`。

第三个限制出现时，先将序列变为 `5 5 5 3 2 2`，但此时第二个限制对应前缀和已经 $>20$，所以可以将后面那一段 `3 2 2` 变为 `2 2 1`（也就是和为 $7$ 的平均分与和为 $5$ 的平均分）。

那么考虑维护这个过程。由于序列长度只有 $10^6$，可以直接用线段树维护这个序列，只需要支持区间推平区间求和。然后也同时维护一个 set 存所有有效限制对应的连续段（就是它所约束着的连续段），这里 set 只需要存区间以及区间和（因为区间内肯定是平均分最优）。第一个部分往前修改时，可以直接从 $x$ 的位置往前遍历 set 的每个连续段，遇到一个合法的段时（即将这段变成新的平均分后满足这段的限制时）就将其删除（限制弱于当前这个），否则就将所有被删除的区间合并为一个新的区间，标记为当前这个限制所约束的区间。

往后修改也只需在 set 上遍历即可，由于颜色段均摊复杂度为 $O(n\log n)$。

code：

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 5e18;
const int N = 1e6+1;
inline ll read() {
	ll s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
int n;
/*
用set维护每段区间：和、区间、贡献
每段内会平均分 
加入一段，首先往前找：
若这段求出来的平均数小于则加上

否则停止，删除扫到的区间，加入新区间。

然后会有一些多余贡献需要往后枚举区间

扫的时候：若可以往后合并，则合并，否则直接将区间和减掉对应数即可
*/
struct node {
	int l,r;
	ll s;
	bool operator < (const node &x) const {
		return l == x.l ? r < x.r : l < x.l;
	}
};
set<node> S;
typedef set<node>::iterator IT;
struct Seg {
	int l,r;
	ll s,tag;
}t[N * 4 + 10];
void build(int x,int l,int r) {
	t[x].l = l, t[x].r = r, t[x].tag = INF;
	if (l == r) return;
	int mid = l + r >> 1;
	build(x<<1,l,mid), build(x<<1|1,mid + 1,r);
}
void f(int x,ll s) {
	t[x].s = (t[x].r - t[x].l + 1ll) * s;
	t[x].tag = s;
}
void pushdown(int x) {
	if (t[x].tag != INF) f(x<<1,t[x].tag), f(x<<1|1,t[x].tag), t[x].tag = INF;
}
void pushup(int x) {
	t[x].s = t[x<<1].s + t[x<<1|1].s;
}
void modify(int x,int l,int r,ll s) {
	if (l > r) return;
	if (t[x].l >= l && r >= t[x].r) {
		f(x,s);
		return;
	}
	pushdown(x);
	int mid = t[x].l + t[x].r >> 1;
	if (mid >= l) modify(x<<1,l,r,s);
	if (mid < r) modify(x<<1|1,l,r,s);
	pushup(x);
}
ll query(int x,int l,int r) {
	if (l > r) return 0;
	if (t[x].l >= l && r >= t[x].r) {
		return t[x].s;
	}
	int mid = t[x].l + t[x].r >> 1;
	pushdown(x);
	ll res = 0;
	if (mid >= l) res = query(x<<1,l,r);
	if (mid < r) res += query(x<<1|1,l,r);
	return res;
}
const int mod = 1e9 + 7;
int qpow(int a,ll b) {
	if (b < 0) return 0;
	int res = 1;
	while (b) {
		if (b & 1) res = 1ll * res * a % mod;
		b >>= 1, a = 1ll * a * a % mod;
	}
	return res;
}
int calc(int x,ll y) {
	if (!y) return 0;
	return (1ll * (x - y % x) * qpow(3,y / x - 1) + y % x * qpow(3,y / x)) % mod;
}
void upd(int l,int r,ll y) {
	int le = r - l + 1;
	modify(1,l,l + y % le - 1,y / le + 1);
	modify(1,l + y % le,r,y / le);
}
int main() {
	n = read();
	int ans = 0;
	S.insert({-1,0,INF}), S.insert({N + 1,N + 1,0});
	S.insert({1,N,0});
	build(1,1,N);
	for (int i = 1;i <= n;i ++ ) {
		int x = read();
		ll y = read(),sum = query(1,1,x);
		if (sum >= y) {
			printf("%d\n",ans);
			continue;
		}
		auto t = --S.lower_bound({x + 1,-1,0ll});
		int l = t -> l,r = t -> r;
		ll ts = t -> s,lv = t -> s / (t -> r - t -> l + 1);
		ans = (ans - calc(r - l + 1,ts) + mod) % mod;
		S.erase(t);
		ll nv = ts - (x - l + 1ll) * lv - min(x - l + 1ll,ts % (r - l + 1));
		if (x != r) S.insert({x + 1,r,nv}), ans = (ans + calc(r - x,nv)) % mod;
		while (1) {
			t = --S.lower_bound({x + 1,-1,0ll});
			if (t -> r == 0) {
				S.insert({1,x,y});
				ans = (ans + calc(x,y)) % mod;
				upd(1,x,y);
				break;
			}
			ll tt = query(1,1,t -> l - 1),vv = y - tt;
			ll lv2 = vv / (x - t -> l + 1);
			if (t -> s > (t -> r - t -> l + 1) * lv2 + min(t -> r - t -> l + 1ll,vv % (x - t -> l + 1))) {
				vv -= t -> s;
				ans = (ans + calc(x - t -> r,vv)) % mod;
				S.insert({t -> r + 1,x,vv});
				upd(t -> r + 1,x,vv);
				break;
			}
			ans = (ans - calc(t -> r - t -> l + 1,t -> s) + mod) % mod;
			S.erase(t);
		}
		ll rest = query(1,1,x) - sum;
		while (rest) {
			t = S.lower_bound({x + 1,-1,0});
			if (t -> r == N + 1) {
				upd(x + 1,N,0);
				S.insert({x + 1,N,0});
				break;
			}
			auto t2 = t;t2 ++;
			ll lv = t -> s / (t -> r - t -> l + 1);
			ll lv2 = (t2 -> s + t2 -> r - t2 -> l) / (t2 -> r - t2 -> l + 1);
			ll all = (t -> r - t -> l + 1ll) * (lv - lv2) + t -> s % (t -> r - t -> l + 1);
			if (rest >= all && t2 -> r != N + 1) {
				ll tv = t -> s + t2 -> s - all;
				int l = t -> l,r = t2 -> r;
				ans = (ans - calc(t -> r - t -> l + 1,t -> s) + mod) % mod;
				ans = (ans - calc(t2 -> r - t2 -> l + 1,t2 -> s) + mod) % mod;
				S.erase(t), S.erase(t2);
				ans = (ans + calc(r - l + 1,tv)) % mod;
				S.insert({l,r,tv});
				rest -= all;
				upd(l,r,tv);
			}
			else {
				ans = (ans - calc(t -> r - t -> l + 1,t -> s) + mod) % mod;
				int l = t -> l,r = t -> r;
				ll ts = max(0ll,t -> s - rest);
				S.erase(t);
				S.insert({l,r,ts}), ans = (ans + calc(r - l + 1,ts)) % mod;
				upd(l,r,ts);
				break;
			}
		}
		printf("%d\n",ans);
	}
	return 0;
}
```
