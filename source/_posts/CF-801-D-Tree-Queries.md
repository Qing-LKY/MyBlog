---
title: CF#801 D.Tree Queries
date: 2022-06-22 16:29:19
tags:
- 换根dp
- 构造
categories: ACM题解
---

原题链接：[https://codeforces.com/contest/1695/problem/D2](https://codeforces.com/contest/1695/problem/D2)

## 题目大意

给你一棵树。要求你指定一组基准点（询问） $\{v_1, v_2, v_3, ... , v_k\}$，对于**树上的每一个点 u**，u 对基准点的最短距离形成的 k 元组 $(d_{u, v_1}, ..., d_{u, v_k})$ 两两不同。问基准点（询问）至少要有多少个。

## 思路分析

这题比赛的时候没想清楚。

首先，一个很明显的结论，如果一个点有 n 个直接儿子，那么至少需要有 n - 1 个询问才能区分它的所有直接儿子。如果只有这个点本身，则不需要添加询问来区分。

于是，我们这样考虑：对于节点 u，我们试图去求区分以 u 的子树内的点需要多少个询问（我们默认 u 会有更上层的点来区分好，因此不去考虑根节点 u，只考虑它的直接和间接子节点）。我们将这个值记为 f[u]。

首先，我们需要区分儿子的子树，因此 f[u] += sum(f[v])。此外，对于 f[v] > 0 的点，意味着这个子节点可以被 u 上面的那个查询和 v 子树内本身已经有的查询来区分。因此，我们记录下 f[v] = 0 的儿子数量 c，我们需要给其中的 c - 1 个儿子进行询问。

故有转移方程：

$f(u) = \sum f(v) + \max (c - 1, 0)$

dp 后得到的 f[u]，是**在默认根节点 u 已经由非子树内节点区分的情况下，区分子树内点所需要的最小询问数**。或者说**默认 u 或 u 的某个父亲上有一个询问（这个询问不记在 f[u] 中，只是假设它存在）的情况下，区分子树内节点的最小询问数**。

也因此，我们最终求出的这个 **f[rt] 还需要加上 1** （也就是根节点上的询问）才是答案。枚举所有的根节点算一遍，取个 min，就是我们要的解。

然后 D1 与 D2 的区别就是换不换根而已了。

换根其实就是在换到 v 前，把当前这个根 u 的值重新算一遍（去掉 v 的影响，因为 v 已经不是 u 的儿子了），然后换到 v 时把 v 的值按着 dp 方程重新算一遍就行。典。具体参考代码。

## 代码

### D1 O(n^2)

```c++
#include <cstdio>
#include <algorithm>
using namespace std;
const int N = 1e5 + 5;
int head[N], nex[N << 1], to[N << 1], n, cc;
inline void addEdge(int u, int v) {
    nex[++cc] = head[u], head[u] = cc, to[cc] = v;
}
int f[N];
void dfs(int u, int fa) {
    int c = 0;
    f[u] = 0;
    for(int i = head[u]; i; i = nex[i]) {
        if(to[i] == fa) continue;
        dfs(to[i], u);
        f[u] += f[to[i]];
        if(!f[to[i]]) c++;
    }
    if(c) f[u] += c - 1;
}
int main() {
    int T; scanf("%d", &T);
    while(T--) {
        scanf("%d", &n); cc = 0;
        for(int i = 1; i <= n; i++) head[i] = 0;
        for(int i = 1; i < n; i++) {
            int u, v; scanf("%d%d", &u, &v);
            addEdge(u, v), addEdge(v, u);
        }
        int res = n - 1;
        for(int i = 1; i <= n; i++) {
            dfs(i, 0);
            res = min(res, f[i] + 1);
        }
        printf("%d\n", res);
    }
    return 0;
}
```

### D2 O(n)

```c++
#include <cstdio>
#include <algorithm>
using namespace std;
const int N = 2e5 + 5;
int head[N], nex[N << 1], to[N << 1], n, cc;
inline void addEdge(int u, int v) {
    nex[++cc] = head[u], head[u] = cc, to[cc] = v;
}
int f[N], res, hav[N];
void dfs1(int u, int fa) {
    int c = 0;
    f[u] = hav[u] = 0;
    for(int i = head[u]; i; i = nex[i]) {
        if(to[i] == fa) continue;
        dfs1(to[i], u);
        f[u] += f[to[i]];
        if(!f[to[i]]) c++;
    }
    if(c) {
        f[u] += c - 1;
        if(c > 1) hav[u] = 1;
    }
}
void dfs2(int u, int fa) {
    if(fa) {
        int c = 0; f[u] = 0;
        for(int i = head[u]; i; i = nex[i]) {
            f[u] += f[to[i]];
            if(!f[to[i]]) c++;
        }
        if(c) f[u] += c - 1;
        if(c > 1) hav[u] = 1; // 记住这个 hav 也是会变的
    }
    int fu = f[u];
    res = min(res, f[u] + 1);
    for(int i = head[u]; i; i = nex[i]) {
        if(to[i] == fa) continue;
        if(f[to[i]]) f[u] = fu - f[to[i]];
        else f[u] = fu - hav[u];
        dfs2(to[i], u);
    }
}
int main() {
    int T; scanf("%d", &T);
    while(T--) {
        for(int i = 1; i <= n; i++) head[i] = 0;
        scanf("%d", &n); cc = 0;
        for(int i = 1; i < n; i++) {
            int u, v; scanf("%d%d", &u, &v);
            addEdge(u, v), addEdge(v, u);
        }
        res = n - 1;
        dfs1(1, 0);
        dfs2(1, 0);
        printf("%d\n", res);
    }
    return 0;
}
```