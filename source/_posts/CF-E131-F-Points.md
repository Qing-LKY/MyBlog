---
title: CF E131 F. Points
date: 2022-07-13 14:41:50
tags:
- 数据结构
- 计数
categories: ACM题解
mathjax: true
---

原题链接：[https://codeforces.com/contest/1701/problem/E](https://codeforces.com/contest/1701/problem/E)

## 题目大意

问你当前序列有多少个三元组满足 $i < j < k$ 且 $k - i \leq d$。$d$ 是一个最开始给定的值。每次操作要求你往序列里（如果不存在）加入 $i$ 或是（如果存在）删去 $i$，然后输出满足要求的三元组的数量。（数都在 $2e5$ 内）

## 思路分析

### 方法一

这个是我看到题之后想到的。

加入一个点时，把这个点的贡献加入答案；删去时，把这个点的贡献从答案中删除。

一个点 $x$ 产生的贡献有三种情况：

1. 这个点是三元组里的 $i$。记与这个点距离小于等于 $d$，且在它的后面的点有 $n$ 个，这种情况的贡献就是 $n\choose 2$。
2. 作为三元组里的 $k$。同理，记与这个点距离小于等于 $d$，且在它的前的点有 $m$ 个，这种情况的贡献就是 $m\choose 2$。
3. 作为三元组的中间点 $j$。它的贡献就是分别处在左右两侧且距离不超过 d 的点对数量。

前两种情形很好处理。用树状数组维护一下点的数量前缀和 $s$ 就行了。

情况三，我们用 $s_i$ 表示 $1$ 到 $i$ 有多少个点（数量前缀和），用 $v_i = s_i - s_{i - 1}$ 表示这个点是否存在。它的贡献是：

$$
\sum^{x-1}_{i=x-d+1} (s_{i+d} - s_{x}) \times v_i
$$

所以我们用树状数组维护 $s_i$，线段树维护区间 $s_{i+d}$ 之和和 $v_i$ 之和（有效点的数量）。

标记的传递、点的有效和无效化、维护的顺序可以参考代码。有亿点小细节，我调了蛮久。

### 方法二

这个做法来自官方题解。个人觉得它更加简洁好写。

我们记每个 $i$ 往后距离不超过 $d$ 的点的数量为 $f_i$，用 $v_i$ 表示 $i$ 上是否有点。显然的，我们的答案是：

$$
\sum^N_{i=1} {f_i\choose 2} \times v_i = \sum^N_{i=1} \frac{f_i \times (f_i - 1)}{2} \times v_i = \sum^N_{i=1} \frac{f_i^2 - f_i}{2} \times v_i
$$

用线段树维护有效的 $\sum f_i^2$ 和 $\sum f_i$ 就行了。

平方和维护的方法有很多种，什么 3*3 的矩阵啊，或是直接按着 $\sum (f_i + t)^2 = \sum f_i^2 + 2tf_i + t^2$ 之类的，这里不再赘述，具体参考代码。 

## 代码

### 方法一

```c++
// 产生一个点的变化时，我们要考虑三个东西：
// 1. 这个点是三元组的起点
// 2. 这个点是三元组的终点
// 3. 这个点是三元组的中点
// 前两者非常好处理，存一下点的数量就行了
// 第三个就是求 \sum (s[i + d] - s[x]) * vis[i]
// 线段树维护一下就行
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;
typedef long long ll;
const int N = 2e5 + 5; //8 + 5;
int q, d, cnt[N];
inline int lowbit(int x) {
    return x & -x;
}
inline void add(int x, int v) {
    while(x <= N - 5) {
        cnt[x] += v;
        x += lowbit(x);
    }
}
inline int sum(int x) {
    int ans = 0;
    while(x) ans += cnt[x], x -= lowbit(x);
    return ans;
}
#define ls (u << 1)
#define rs ((u << 1) | 1)
ll tr[N << 2], tag[N << 2], siz[N << 2];
inline void maintain(int u) {
    tr[u] = tr[ls] + tr[rs];
    siz[u] = siz[ls] + siz[rs];
}
inline void upd(int u, ll d) {
    tr[u] += d * siz[u];
    tag[u] += d;
}
inline void pushdown(int u) {
    if(tag[u]) {
        upd(ls, tag[u]), upd(rs, tag[u]);
        tag[u] = 0;
    }
}
void update(int u, int l, int r, int ql, int qr, ll d) {
    if(r < ql || l > qr) return;
    if(ql <= l && r <= qr) return upd(u, d);
    int mid = (l + r) >> 1;
    pushdown(u);
    update(ls, l, mid, ql, qr, d), update(rs, mid + 1, r, ql, qr, d);
    maintain(u);
}
void edit(int u, int l, int r, int x) {
    if(r < x || l > x) return;
    if(l == r) {
        siz[u] ^= 1;
        tr[u] = siz[u] * sum(min(x + d, N - 5));
        return;
    }
    int mid = (l + r) >> 1;
    pushdown(u);
    edit(ls, l, mid, x), edit(rs, mid + 1, r, x);
    maintain(u);
}
ll query(int u, int l, int r, int ql, int qr) {
    if(r < ql || l > qr) return 0;
    if(ql <= l && r <= qr) return tr[u];
    int mid = (l + r) >> 1;
    pushdown(u);
    return query(ls, l, mid, ql, qr) + query(rs, mid + 1, r, ql, qr);
}
void debug(int u, int l, int r) {
    if(l == r) {
        printf("i=%d s[i+d]=%lld siz[i]=%lld\n", l, tr[u], siz[u]);
        return;
    }
    int mid = (l + r) >> 1;
    pushdown(u);
    debug(ls, l, mid), debug(rs, mid + 1, r);
}
int vis[N];
int main() {
    memset(vis, -1, sizeof(vis));
    scanf("%d%d", &q, &d);
    ll res = 0;
    for(int i = 1; i <= q; i++) {
        int x; scanf("%d", &x);
        vis[x] *= -1;
        // add(x, vis[x]); 不能在这里更新树状数组
        // 作为起点
        ll t = sum(min(x + d, N - 5)) - sum(x), t2;
        res += vis[x] * t * (t - 1) / 2;
        // 作为终点
        t = sum(x - 1) - sum(max(0, x - d - 1)); //你要求的是 [x-d, x-1]！
        res += vis[x] * t * (t - 1) / 2;
        // 作为中点
        t = query(1, 1, N - 5, max(1, x - d + 1), x - 1);
        // t2使用的是树状数组里的数据 t1使用的是线段树里的数据 我们需要保证这两者是同步的
        // 因此 要么一起放到前面更新 要么一起放到后面更新
        t2 = (ll)(sum(x - 1) - sum(max(x - d, 0))) * sum(x); // 这里要记得开ll！
        res += vis[x] * (t - t2);
        // 输出答案
        printf("%lld\n", res);
        // 更新 S[i+d] 的维护
        add(x, vis[x]);
        update(1, 1, N - 5, max(x - d, 1), N - 5, vis[x]);
        edit(1, 1, N - 5, x); // 注意这个必须在后面 不然会被额外upd一次
        /*if(i >= 5) {
            debug(1, 1, N - 5);
            for(int j = 1; j <= N - 5; j++) printf("%d(%d) ", j, sum(j) - sum(j - 1));
            puts("");
        }*/
    }
    return 0;
}
```

### 方法二

```c++
// ai^2 + 2xai + x^2
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;
typedef long long ll;
const int N = 2e5 + 5;
ll tr[2][N << 2], tag[N << 2], siz[N << 2];
int m, cnt[N];
inline int lowbit(int x) {
    return x & -x;
}
inline void add(int x, int d) {
    while(x <= N - 5) {
        cnt[x] += d;
        x += lowbit(x);
    }
}
inline int sum(int x) {
    int an = 0;
    while(x) an += cnt[x], x -= lowbit(x);
    return an;
}
#define ls (u << 1)
#define rs ((u << 1) | 1)
inline void maintain(int u) {
    tr[0][u] = tr[0][ls] + tr[0][rs];
    tr[1][u] = tr[1][ls] + tr[1][rs];
    siz[u] = siz[ls] + siz[rs];
}
inline void add(int u, ll d) {
    tr[1][u] += 2ll * d * tr[0][u] + d * d * siz[u];
    tr[0][u] += d * siz[u];
    tag[u] += d;
}
inline void pushdown(int u) {
    if(tag[u]) {
        add(ls, tag[u]), add(rs, tag[u]);
        tag[u] = 0;
    }
}
void update(int u, int l, int r, int ql, int qr, ll d) {
    if(r < ql || l > qr) return;
    if(ql <= l && r <= qr) return add(u, d);
    int mid = (l + r) >> 1;
    pushdown(u);
    update(ls, l, mid, ql, qr, d), update(rs, mid + 1, r, ql, qr, d);
    maintain(u);
}
void edit(int u, int l, int r, int x, ll d) {
    if(r < x || l > x) return;
    if(l == r) {
        siz[u] += d;
        ll fi = sum(min(N - 5, x + m)) - sum(x);
        tr[0][u] += d * fi;
        tr[1][u] += d * fi * fi;
        return; 
    }
    int mid = (l + r) >> 1;
    pushdown(u);
    edit(ls, l, mid, x, d), edit(rs, mid + 1, r, x, d);
    maintain(u);
}
ll getAns() {
    return (tr[1][1] - tr[0][1]) / 2;
}
int vis[N];
int main() {
    memset(vis, -1, sizeof(vis));
    int q, d;
    scanf("%d%d", &q, &d);
    m = d;
    for(int i = 1; i <= q; i++) {
        int x; scanf("%d", &x);
        vis[x] *= -1;
        add(x, vis[x]);
        edit(1, 1, N - 5, x, vis[x]);
        update(1, 1, N - 5, max(1, x - d), x - 1, vis[x]);
        printf("%lld\n", getAns());
    }
    return 0;
}
```