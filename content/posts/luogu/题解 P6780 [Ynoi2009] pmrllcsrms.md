---
title: "题解 P6780 [Ynoi2009] pmrllcsrms"
date: 2023-08-02T10:19:00+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-p6780-ynoi2009-pmrllcsrms"
original: "https://www.luogu.com.cn/article/weqvgnmp"
math: true
---
第一个手搓 7k 代码。

图片来自：https://www.luogu.com.cn/blog/111055/how-to-set-a-cool-url

对序列按 $C$ 分块，块内是最大子段和，块间可以看成第 $i$ 块的后缀加上 $i+1$ 块的前缀。

那么就是要求满足长度之和 $\le C$ 的后缀加前缀的最大值。

发现这就是一个直角三角形中求最大值。

![](/images/luogu/23pxy9vb.png)

那么对于一个大的等腰直角三角形，可以把它分成 $3$ 部分（如图），上面和右边可以递归下去查询，中间的部分则是两个区间最大值之和。

![](/images/luogu/yjz0ounw.png)

此时 $mid$ 区域的答案为这个矩形长对应的区间中 $B_y$ 的最大值和宽对应的区间中 $A_x$ 的最大值之和。

那么这个三角形的信息是可以合并的，线段树维护每个三角形对应区间中 $A,B$ 的最大值以及答案即可（这里横坐标的一个区间可以代表一个斜边贴在大三角形斜边上，底边为对应区间的等腰直角三角形的信息）。

最后由于块长不定，我们需要动态维护区间的整块的答案，可以再开个线段树维护块 $[l,r]$ 的答案（即每个块最大子段和以及每相邻两个块的答案取最大值）。

对于两边的散块，在直角三角形里对应的就是不完整的一部分，可以用如图的方式拆成若干部分，其中三角形的部分可以区间查询，矩形的部分就求求最大值然后加一加即可。
![](/images/luogu/6jlqhcnb.png)
现在考虑修改。单点的修改造成的影响有：

- 最大子段和线段树中的单点修改。
- 块间答案的区间加。（对于 $A$ 或者 $B$ 的区间加，直接打 tag 维护即可，注意序列区间在三角形上对应的区间略有不同。）
- 整块区间答案的单点修改。

注意最后一个块是不完整的，比较麻烦。省事的方法是将序列补成长度为 $C$ 的倍数，但是较慢。也可以单独判判。

$O(m\log n)$。可能略微卡常，开快读即可。
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 4e6;
const int B = 2e6;
namespace IO{//by cyffff
    int len=0;
    char ibuf[(1<<25)+1],*iS,*iT,out[(1<<23)+1];
    #if ONLINE_JUDGE
    #define gh() (iS==iT?iT=(iS=ibuf)+fread(ibuf,1,(1<<20)+1,stdin),(iS==iT?EOF:*iS++):*iS++)
    #else
    #define gh() getchar()
    #endif
    #define reg register
    inline ll read(){
        reg char ch=gh();
        reg ll x=0;
        reg char t=0;
        while(ch<'0'||ch>'9')   t|=ch=='-',ch=gh();
        while(ch>='0'&&ch<='9') x=x*10+(ch^48),ch=gh();
        return t?-x:x;
    }
    inline void putc(char ch){
        out[len++]=ch;
    }
    template<class T>
    inline void write(T x){
        if(x<0)putc('-'),x=-x;
        if(x>9)write(x/10);
        out[len++]=x%10+48;
    }
    inline void flush(){
        fwrite(out,1,len,stdout);
        len=0;
    }
    inline char getc(){
        char ch=gh();
        while(ch<'A'||ch>'Z') ch=gh();
        return ch;
    }
}
using IO::read;
using IO::write;
using IO::flush;
using IO::getc;
using IO::putc;

int n,m,C;
int st[B + 10],ed[B + 10],bl[N + 10],a[N + 10];
ll pr[N + 10],su[N + 10];
struct Segment_Tree1 {
	struct node {
		ll ls,rs,sum,s;
		inline friend node operator + (node a,node b) {
			node c;
			c.sum = a.sum + b.sum;
			c.ls = max(a.ls,a.sum + b.ls);
			c.rs = max(b.rs,b.sum + a.rs);
			c.s = max({a.s,b.s,a.rs + b.ls});
			return c;
		}
	};
	struct Seg {
		int l,r;
		node v;
	}t[N * 4 + 10];
	inline void pushup(int x) {
		t[x].v = t[x<<1].v + t[x<<1|1].v;
	}
	void build(int x,int l,int r) {
		t[x].l = l, t[x].r = r;
		if (l == r) {
			t[x].v.ls = t[x].v.rs = t[x].v.s = t[x].v.sum = a[l];
			return;
		}
		int mid = l + r >> 1;
		build(x<<1,l,mid), build(x<<1|1,mid + 1,r);
		pushup(x);
	}
	void modify(int x,int k,int s) {
		if (t[x].l == t[x].r) {
			t[x].v.ls = t[x].v.rs = t[x].v.s = t[x].v.sum = s;
			return;
		}
		int mid = t[x].l + t[x].r >> 1;
		if (mid >= k) modify(x<<1,k,s);
		else modify(x<<1|1,k,s);
		pushup(x);
	}
	node query(int x,int l,int r) {
		if (t[x].l >= l && r >= t[x].r) return t[x].v;
		int mid = t[x].l + t[x].r >> 1;
		if (mid < l) return query(x<<1|1,l,r);
		if (mid >= r) return query(x<<1,l,r);
		return query(x<<1,l,r) + query(x<<1|1,l,r);
	}
}T1;
struct Node {
	ll sx,sy,s;
	//sx:竖着的区间mx
	//sy:哼着的区间mx 
	inline friend Node operator + (Node a,Node b) {
		return {max(a.sx,b.sx),max(a.sy,b.sy),max({a.s,b.s,a.sx + b.sy})};
	}
};
struct Segment_Tree2 {
	int pos;//pos&&pos+1
	struct Seg {
		int l,r;
		ll tag[2];
		Node v;
	};
	vector<Seg> t;
	inline void pushup(int x) {
		t[x].v = t[x<<1].v + t[x<<1|1].v;
	}
	void build(int x,int l,int r) {
		t[x].l = l, t[x].r = r;
		if (l == r) {
			t[x].v.sx = !l ? 0 : su[st[pos] + l - 1], t[x].v.sy = l >= ed[pos + 1] - st[pos + 1] + 1 ? 0 : pr[ed[pos + 1] - l];
			t[x].v.s = t[x].v.sx + t[x].v.sy;
			return;
		}
		int mid = l + r >> 1;
		build(x<<1,l,mid), build(x<<1|1,mid+1,r);
		pushup(x);
	}
	inline void f(int x,ll *s) {
		t[x].v.sx += s[0], t[x].v.sy += s[1];
		t[x].tag[0] += s[0], t[x].tag[1] += s[1];
		t[x].v.s += s[0] + s[1];
	}
	inline void pushdown(int x) {
		f(x<<1,t[x].tag), f(x<<1|1,t[x].tag);
		t[x].tag[0] = t[x].tag[1] = 0;
	}
	void modify(int x,int l,int r,ll *s) {
		if (t[x].l >= l && r >= t[x].r) {
			f(x,s);
			return;
		}
		int mid = t[x].l + t[x].r >> 1;
		pushdown(x);
		if (mid >= l) modify(x<<1,l,r,s);
		if (mid < r) modify(x<<1|1,l,r,s);
		pushup(x);
	}
	Node query(int x,int l,int r) {
		if (l > r) return {-INF,-INF};
		if (t[x].l >= l && r >= t[x].r) return t[x].v;
		pushdown(x);
		int mid = t[x].l + t[x].r >> 1;
		if (mid < l) return query(x<<1|1,l,r);
		if (mid >= r) return query(x<<1,l,r);
		return query(x<<1,l,r) + query(x<<1|1,l,r);
	}
}T2[B + 10];
struct Segment_Tree3 {
	//块 l ~ r 
	struct Seg {
		int l,r;
		ll mx1,mx2;
	}t[B * 4 + 10];
	inline void pushup(int x) {
		t[x].mx1 = max(t[x<<1].mx1,t[x<<1|1].mx1);
		t[x].mx2 = max(t[x<<1].mx2,t[x<<1|1].mx2);
	}
	void build(int x,int l,int r) {
		t[x].l = l, t[x].r = r;
		if (l == r) {
			t[x].mx1 = T1.query(1,st[l],ed[l]).s;
			if (l != bl[n]) t[x].mx2 = T2[l].t[1].v.s;
			return;
		}
		int mid = l + r >> 1;
		build(x<<1,l,mid), build(x<<1|1,mid + 1,r);
		pushup(x);
	}
	void modify(int x,int k,int fl,ll s) {
		if (t[x].l == t[x].r) {
			fl ? t[x].mx2 = s : t[x].mx1 = s;
			return;
		}
		int mid = t[x].l + t[x].r >> 1;
		if (mid >= k) modify(x<<1,k,fl,s);
		else modify(x<<1|1,k,fl,s);
		pushup(x);
	}
	ll query(int x,int l,int r,int fl) {
		if (l > r) return -INF;
		if (t[x].l >= l && r >= t[x].r) {
			return fl ? t[x].mx2 : t[x].mx1;
		}
		int mid = t[x].l + t[x].r >> 1;
		ll res = -INF;
		if (mid >= l) res = query(x<<1,l,r,fl);
		if (mid < r) res = max(res,query(x<<1|1,l,r,fl));
		return res;
	}
}T3;
void build() {
	int cnt = 0;
	for (int i = 1;i <= n;i += C) {
		st[++cnt] = i, ed[cnt] = min(i + C - 1,n);
		for (int j = st[cnt];j <= ed[cnt];j ++ ) 
			bl[j] = cnt, pr[j] = j == st[cnt] ? a[j] : pr[j - 1] + a[j];
		for (int j = ed[cnt];j >= st[cnt];j -- ) 
			su[st[cnt] + ed[cnt] - j] = j == ed[cnt] ? a[j] : su[st[cnt] + ed[cnt] - j - 1] + a[j];
	}
	T1.build(1,1,n);
	for (int i = 1;i < bl[n];i ++ ) T2[i].t.resize(C * 4 + 1), T2[i].pos = i, T2[i].build(1,0,C);
	T3.build(1,1,bl[n]);
}
int main() {
	n = read(), m = read(), C = read();
	for (int i = 1;i <= n;i ++ ) a[i] = read();
	while (n % C) n ++;
	build();
	while (m -- ) {
		int op = read();
		if (op == 1) {
			int x = read(),y = read(),l = bl[x];
			T1.modify(1,x,y);
			T3.modify(1,l,0,T1.query(1,st[l],ed[l]).s);
			ll tmp[2] = {};
			tmp[0] = y - a[x];
			if (l < bl[n])
				T2[l].modify(1,ed[l] - x + 1,C,tmp),
				T3.modify(1,l,1,T2[l].t[1].v.s);
			swap(tmp[0],tmp[1]);
			if (l > 1)
				T2[l - 1].modify(1,0,C - (x - st[l] + 1),tmp),
				T3.modify(1,l - 1,1,T2[l - 1].t[1].v.s);
			a[x] = y;
		}
		else {
			int x = read(),y = read(),l = bl[x],r = bl[y];
			int l1 = ed[l] - x + 1,l2 = y - st[r] + 1;
			if (l == r || y - x + 1 <= C) {
				printf("%lld\n",max(0ll,T1.query(1,x,y).s));
				continue;
			}
			ll ans = max({0ll,T1.query(1,x,ed[l]).s,T1.query(1,st[r],y).s});
			if (l + 1 == r) {
				ll mx1 = T2[l].query(1,0,C - l2).sx;
				Node tmp = T2[l].query(1,C - l2,l1);
				ll mx2 = tmp.sy;
				ll mx3 = tmp.sx;
				ll mx4 = T2[l].query(1,l1+1,C).sy;
				printf("%lld\n",max({ans,mx1+mx2,mx1+mx4,mx3+mx4,tmp.s}));
				continue;
			}
			ans = max({ans,T3.query(1,l + 1,r - 2,1),T3.query(1,l + 1,r - 1,0)});
			Node tmp1 = T2[l].query(1,0,l1);
			ans = max({ans,tmp1.s,tmp1.sx + T2[l].query(1,l1,C).sy});
			Node tmp2 = T2[r - 1].query(1,C - l2,C);
			ans = max({ans,tmp2.s,tmp2.sy + T2[r - 1].query(1,0,C - l2).sx});
			printf("%lld\n",max(ans,0ll));
		}
	}
	return 0;
}
/*
9 2 3
1 -3 4 5 -3 8 -6 -9 10
1 7 6
2 3 8

5 2 2
1 -2 3 -4 5
1 4 1
2 3 5
*/
```
