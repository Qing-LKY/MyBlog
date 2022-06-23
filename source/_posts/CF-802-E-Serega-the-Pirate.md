---
title: CF#802 E.Serega the Pirate
date: 2022-06-23 10:39:50
tags:
- 构造
- 暴力
categories: ACM题解
mathjax: true
---

原题链接：[https://codeforces.com/contest/1700/problem/E](https://codeforces.com/contest/1700/problem/E)

## 题目大意

给你一个 n*m 的矩阵，矩阵上的元素是 nm 的排列。要求：能够找到一条路径遍历所有的点（每个点可经过多次），记每个点在第 xi 步第一次被访问，要求有 x1 < x2 < x3 ...。现在问你：原本的矩阵是否满足要求？如果不满足，能否通过1次交换使得它满足要求，可能的交换种类有多少种？

## 思路分析

一道简单的暴力题。比赛时没想清楚。

一个很明显的结论：1 肯定是起点，对于每个大于 1 的点，它的周围至少要有一个小于它的点。如果所有的点都可以满足这个要求，那么矩阵满足要求。

再考虑交换会发生什么：它只会改变交换的点自身和它周围的点的这个状态。

这意味着：我们的交换至少要包含一个不满足要求的点（或是那个点周围的点），每次交换最多只会影响 10 个点（两个被交换点和它的相邻点）的状态。

所以，我们找到一个不满足要求的点之后，直接枚举它和它周围的这 5 个点作为其中一个交换点，然后枚举所有点作为另外一个交换点，动态地维护、计算图上有多少个点满足要求就行了。复杂度大概是 $5 * n * m * 10 * 4 = O(n * m)$ 的。

去重什么的就自己看着办就行了。我是存起来排序 unique 的。直接记录然后减也是可以的。

## 代码

```c++
#include <cstdio>
#include <algorithm>
using namespace std;
const int N = 4e5 + 5;
const int zl[5][2] = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}, {0, 0}};
int n, m, A[N], f[N], tot, cc;
inline int id(int x, int y) {
    return x * m + y;
}
inline int isBad(int x, int y) {
    if(A[id(x, y)] == 1) return 0;
    int v = A[id(x, y)];
    if(x > 0 && A[id(x - 1, y)] < v) return 0;
    if(y > 0 && A[id(x, y - 1)] < v) return 0;
    if(x < n - 1 && A[id(x + 1, y)] < v) return 0;
    if(y < m - 1 && A[id(x, y + 1)] < v) return 0;
    return 1;
}
inline bool inSqu(int x, int y) {
    return x >= 0 && x < n && y >= 0 && y < m;
}
bool vis[N];
inline bool judge(int x, int y, int p, int q) {
    int i1 = id(x, y), i2 = id(p, q), del = 0;
    int tx, ty, d, i3;
    swap(A[i1], A[i2]);
    for(int i = 0; i < 5; i++) {
        // for i1
        tx = x + zl[i][0], ty = y + zl[i][1];
        if(inSqu(tx, ty)) {
            i3 = id(tx, ty);
            if(!vis[i3]) {
                vis[i3] = 1, d = isBad(tx, ty);
                if(d ^ f[i3]) {
                    if(d > 0) del--;
                    else del++;
                }
            }
        }
        // for i2
        tx = p + zl[i][0], ty = q + zl[i][1];
        if(!inSqu(tx, ty)) continue;
        i3 = id(tx, ty);
        if(!vis[i3]) {
            vis[i3] = 1, d = isBad(tx, ty);
            if(d ^ f[i3]) {
                if(d > 0) del--;
                else del++;
            }
        }
    }
    // clear
    for(int i = 0; i < 5; i++) {
        // for i1
        int tx = x + zl[i][0], ty = y + zl[i][1];
        if(inSqu(tx, ty)) vis[id(tx, ty)] = 0;
        // for i2
        tx = p + zl[i][0], ty = q + zl[i][1];
        if(!inSqu(tx, ty)) continue; // 注意上面不能 continue 哦，因为下面的还要算的
        vis[id(tx, ty)] = 0;
    }
    swap(A[i1], A[i2]);
    return del == tot;
}
struct Op {
    int a, b;
    Op(int s = 0, int t = 0): a(s), b(t) {
        if(a > b) swap(a, b);
    }
    bool operator < (const Op &B) const {
        if(a != B.a) return a < B.a;
        return b < B.b;
    }
    bool operator == (const Op &B) const {
        return a == B.a && b == B.b;
    }
}L[N * 5];
int main() {
    scanf("%d%d", &n, &m);
    for(int i = 0; i < n * m; i++) scanf("%d", A + i);
    int bx = -1, by = -1;
    for(int i = 0; i < n; i++) {
        for(int j = 0; j < m; j++) {
            if(isBad(i, j)) {
                tot += (f[i * m + j] = 1);
                bx = i, by = j;
            }
        }
    }
    if(bx == -1) return puts("0"), 0;
    for(int i = 0; i < 5; i++) {
        int x = bx + zl[i][0], y = by + zl[i][1];
        if(x < 0 || x >= n || y < 0 || y >= m) continue;
        for(int p = 0; p < n; p++) {
            for(int q = 0; q < m; q++) {
                if(judge(x, y, p, q)) {
                    // printf("(%d, %d) - (%d, %d)\n", x, y, p, q);
                    L[++cc] = Op(id(x, y), id(p, q));
                }
            }
        }
    }
    if(cc == 0) return puts("2"), 0;
    sort(L + 1, L + cc + 1);
    cc = unique(L + 1, L + cc + 1) - L - 1;
    printf("1 %d\n", cc);
    return 0;
}
```