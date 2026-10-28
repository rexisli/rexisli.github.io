---
title: 主席树（单点修改，区间第 k 小）
date: 2026-10-27 23:41:00
categories:
  - 模板
  - 线段树
tags:
  - 主席树
  - 可持久化线段树
  - 第k小
math: true
---

## 摘要

静态区间第 $k$ 小看上去是二维问题（位置区间 × 值域），但它有一个几乎免费的降维方式：把序列的**每个前缀**建成一棵值域线段树 $V_i$，其中结点存「前 $i$ 个元素落在该值域区间的个数」。那么区间 $[l,r]$ 的计数就是**两个版本的差** $V_r-V_{l-1}$，第 $k$ 小只要在这棵「虚拟树」上二分。

难点只在空间：每个前缀都建一整棵树是 $O(n^2)$。但相邻前缀只差一个元素，因此只需复制根到叶的 $O(\log n)$ 个结点，其余子树与旧版本共享——这就是**路径复制（path copying）**。本文给出可减性引理、路径复制不变量、带归纳证明的查询正确性，以及内存与下标的常见坑。

## 1 问题与记号

给定 $a_1,\dots,a_n$，回答 $q$ 次询问：区间 $[l,r]$ 中第 $k$ 小的数。

先把值域离散化到 $[1,m]$（$m\le n$）。记

$$c_i(v)=\#\{j\le i:a_j=v\},\qquad C_{l,r}(S)=\sum_{v\in S}c_r(v)-c_{l-1}(v).$$

$C_{l,r}(S)$ 就是「$[l,r]$ 中落在值域集合 $S$ 的元素个数」。数据结构只需要能算它，就能二分。

## 2 核心思想：版本即前缀和

**引理 1（可减性）.** 对任意值域区间 $S$，$C_{l,r}(S)=\mathrm{dat}_r(S)-\mathrm{dat}_{l-1}(S)$，其中 $\mathrm{dat}_i(S)$ 是版本 $V_i$ 中对应 $S$ 的结点的值。

*证明.* 计数是逐元素相加的：$\sum_{j\le r}[a_j\in S]-\sum_{j\le l-1}[a_j\in S]=\sum_{l\le j\le r}[a_j\in S]$。$\square$

这就是全部思想：普通前缀和的前缀是**数**，这里的前缀是**整棵树**；而「相减」这个操作在树上逐结点仍然成立。于是区间问题被化成两个版本之差，值域二分则变成在两棵树的**同一位置**同时向下走。

## 3 路径复制

建版本 $V_i$：从 $V_{i-1}$ 出发，在位置 $a_i$ 上把计数加 1。根到叶子这条链上的结点都要变，其余不变。做法是：

1. 走到一个结点，先 @@clone_node@@ 复制一份（新下标），得到可写副本；
2. 递归进对应儿子，把返回的新儿子下标写进副本；
3. @@push_up@@ 重算副本的 @@dat@@。

未被访问的子树保持旧下标，于是**旧版本完全不受影响**。

**引理 2（新增结点数）.** 版本 $V_i$ 相对 $V_{i-1}$ 新增 $O(\log_2 n)$ 个结点（恰为根到叶的层数）。

**引理 3（结构共享安全）.** 若所有写操作只作用于 @@create_node/clone_node@@ 产生的新结点，则任何旧版本可达的结点值不变。

*证明.* 旧版本的可达集合是根的下标链；写操作只改新下标结点或从旧结点复制的字段，从不写旧下标结点。$\square$

注意引理 3 的措辞：**共享是安全的，条件是「不可变」**。一旦某处直接修改了旧结点（例如懒标记下传时改子结点），可持久化立刻失效——下一篇正是踩这个坑。

## 4 正确性

**不变量 I.** 版本 $V_i$ 中任一结点对应值域区间 $S$，其 @@dat@@ 等于 $C_{1,i}(S)$。

**引理 4（建树）.** @@build@@ 自底向上按「叶子取 $arr[值]$、内部取左右和」构造，得 $V$ 表示整个数组的计数树，满足 I（此处对应「版本 $n$」，作为初始版本 0 的等价物）。

**引理 5（单点加保持 I）.** @@add@@ 只沿一条链复制并重算 @@dat@@ = 左子树和 + 右子树和；未访问子树仍是上一版本的值，故新版本满足 I。

**定理（查询）.** @@query_kth(ver_l, ver_r, k)@@ 在 $[1,\min(k,r-l+1)]$ 上返回区间第 $k$ 小的离散化下标。

*证明.* 对值域长度归纳。当前结点值域 $[L,R]$，令左儿子值域 $[L,M]$，

$$\mathrm{cnt}=\mathrm{dat}_{r}([L,M])-\mathrm{dat}_{l-1}([L,M])$$

由引理 1 即 $C_{l,r}([L,M])$，是区间中落在左半值域的元素个数。若 $k\le\mathrm{cnt}$，第 $k$ 小必在左半，问题化为同构子问题；否则答案在右半且为右半的第 $k-\mathrm{cnt}$ 小。$L=R$ 时唯一确定。$\square$

注意代码里 @@dat@@ 在本文语境下是**计数**，不是区间和——同一个字段名在不同模板里含义不同，读代码时以 @@push_up@@ 的定义为准。

## 5 复杂度

| 项 | 复杂度 | 备注 |
| --- | --- | --- |
| 建树 | $O(n)$ | 一次自底向上 |
| 每版本 | $O(\log n)$ | 路径长度 |
| 每次查询 | $O(\log n)$ | 值域二分 |
| 空间 | $O(n\log n)$ | 约 $n(\log_2 n+1)$ 个结点 |

$n=2\times10^5$、$m=2\times10^5$ 时结点数约 $4\times10^6$。结点含三个 @@int@@ 时约 48 MB，配合 @@vector@@ 的扩容拷贝会翻倍——**务必 @@reserve@@**。

## 6 实现

```cpp
class PresidentSegmentTree {
	struct node {
		int lc = -1, rc = -1;
		int dat = 0;
	};

	vector<node> tr;
	vector<int> root;
	int rangeL, rangeR;

	inline int create_node() {
		tr.emplace_back();
		return tr.size() - 1;
	}

	inline int clone_node(int rt) {
		if (rt == -1) return create_node();
		tr.emplace_back(tr[rt]);
		return tr.size() - 1;
	}

	inline void push_up(int rt) {
		int res = 0;
		res += (tr[rt].lc != -1 ? tr[tr[rt].lc].dat : 0);
		res += (tr[rt].rc != -1 ? tr[tr[rt].rc].dat : 0);
		tr[rt].dat = res;
	}

	inline int apply_assign(int rt, int l, int r, int val) {
		if (rt == -1) rt = create_node();
		tr[rt].dat = val * (r - l + 1);
		return rt;
	}

	inline int apply_add(int rt, int l, int r, int val) {
		if (rt == -1) rt = create_node();
		tr[rt].dat += val * (r - l + 1);
		return rt;
	}

	int build(int rt, int l, int r, const vector<int> &arr) {
		if (rt == -1) rt = create_node();
		if (l == r) {
			tr[rt].dat = arr[l];
			return rt;
		}
		int mid = (l + r) >> 1;
		tr[rt].lc = build(tr[rt].lc, l, mid, arr);
		tr[rt].rc = build(tr[rt].rc, mid + 1, r, arr);
		push_up(rt);
		return rt;
	}

	int assign(int rt, int l, int r, int lb, int ub, int val) {
		rt = clone_node(rt);
		if (lb <= l && r <= ub) {
			rt = apply_assign(rt, l, r, val);
			return rt;
		}
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = assign(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub)  tr[rt].rc = assign(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt);
		return rt;
	}

	int add(int rt, int l, int r, int lb, int ub, int val) {
		rt = clone_node(rt);
		if (lb <= l && r <= ub) {
			rt = apply_add(rt, l, r, val);
			return rt;
		}
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = add(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub)  tr[rt].rc = add(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt);
		return rt;
	}

	int query(int rt, int l, int r, int lb, int ub) {
		if (rt == -1) return 0;
		if (lb <= l && r <= ub) return tr[rt].dat;
		int ans = 0;
		int mid = (l + r) >> 1;
		if (lb <= mid) ans += query(tr[rt].lc, l, mid, lb, ub);
		if (mid < ub)  ans += query(tr[rt].rc, mid + 1, r, lb, ub);
		return ans;
	}

	int query_kth(int ver_l, int ver_r, int l, int r, int pos) {
		if (l == r) return l;
		int lc_l = (ver_l != -1 ? tr[ver_l].lc : -1);
		int lc_r = (ver_r != -1 ? tr[ver_r].lc : -1);
		int rc_l = (ver_l != -1 ? tr[ver_l].rc : -1);
		int rc_r = (ver_r != -1 ? tr[ver_r].rc : -1);
		int cnt = (lc_r != -1 ? tr[lc_r].dat : 0) - (lc_l != -1 ? tr[lc_l].dat : 0);
		int mid = (l + r) >> 1;
		if (pos <= cnt) return query_kth(lc_l, lc_r, l, mid, pos);
		else return query_kth(rc_l, rc_r, mid + 1, r, pos - cnt);
	}

public:
	PresidentSegmentTree() : rangeL(1), rangeR(0) {
		tr.emplace_back();
		root.emplace_back(-1);
	}

	explicit PresidentSegmentTree(int l, int r) : rangeL(l), rangeR(r) {
		tr.emplace_back();
		root.emplace_back(-1);
	}

	explicit PresidentSegmentTree(int n) : PresidentSegmentTree(1, n) {
  }

	inline void clear() {
		rangeL = 1, rangeR = 0;
		tr.clear(), tr.emplace_back();
		root.clear();
		root.emplace_back();
	}

	inline void reset(int l, int r) {
		clear();
		rangeL = l, rangeR = r;
	}

	inline void build(const vector<int>& arr, int l, int r) {
		reset(l, r);
		root.clear();
		root.emplace_back(build(-1, rangeL, rangeR, arr));
	}

	inline void assign(int ver, int pos, int val) {
		assert(rangeL <= rangeR);
		root.emplace_back(assign(root[ver], rangeL, rangeR, pos, pos, val));
	}

	inline void add(int ver, int pos, int val) {
		assert(rangeL <= rangeR);
		root.emplace_back(add(root[ver], rangeL, rangeR, pos, pos, val));
	}

	inline int query(int ver, int pos) {
		assert(rangeL <= rangeR);
		return query(root[ver], rangeL, rangeR, pos, pos);
	}

	inline int query_kth(int ver_l, int ver_r, int pos) {
		assert(rangeL <= rangeR);
		return query_kth(root[ver_l], root[ver_r], rangeL, rangeR, pos);
	}
};
```

## 7 使用

```cpp
int n, q;
cin >> n >> q;
vector<int> a(n + 1), b(n);
for (int i = 1; i <= n; i++) cin >> a[i], b[i - 1] = a[i];

// 离散化，把值域压到 [1, n]
sort(b.begin(), b.end());
b.erase(unique(b.begin(), b.end()), b.end());
for (int i = 1; i <= n; i++)
    a[i] = lower_bound(b.begin(), b.end(), a[i]) - b.begin() + 1;

PresidentSegmentTree pst(n);
pst.build(a, 1, n);                 // 版本 0

for (int i = 1; i <= n; i++)
    pst.add(i - 1, a[i], 1);        // 版本 i：在版本 i-1 上把 a[i] 计数 +1

int l, r, k;
cin >> l >> r >> k;
int pos = pst.query_kth(l - 1, r, k);  // 返回离散化后的下标
cout << b[pos - 1] << "\n";           // 换回原值
```

## 8 实现细节与易错点

- **@@ver_l@@ 传 $l-1$**，不是 $l$。传 $l$ 等于丢掉第一个元素，答案会偏大，且只有 $k$ 恰好落在被丢元素上时才看得出来。
- **离散化下标必须 1-based**：@@lower_bound(...)-begin()+1@@。用 0-based 时 $arr[0]$ 位置空着、值域下界对不上 @@rangeL@@。
- **@@rangeL/rangeR@@ 用离散化后的值域** $[1,m]$（$m$ 为去重后个数），不是 $[1,n]$。
- **@@k@@ 的下标语义**：$k\in[1,r-l+1]$，从 1 开始；越界会一直走到叶子给出错误答案，建议 @@assert@@。
- **返回的是离散化下标**，要 @@b[pos-1]@@ 换回原值。
- **第 $k$ 大 = 第 $r-l+2-k$ 小**，别重复写一套。
- **@@push_up@@ 要处理 $-1$ 空儿子**：@@tr[-1]@@ 是未定义行为，代码里用三目先判 @@!= -1@@。
- **@@vector@@ 引用失效**：@@emplace_back@@ 可能重分配，别跨它持有 @@node&@@。
- **$n=1$ 或元素全相同**（$m=1$）：只有一层，@@query_kth@@ 直接返回 $1$，注意别写死循环。

## 9 扩展

**带修区间第 $k$ 小.** 在序列上套树状数组，每个 BIT 结点挂一棵动态开点线段树；修改要动 $O(\log n)$ 棵树，查询把 $O(\log n)$ 个根同时推进。这是 OI 的经典 P2617。

**区间不同数个数.** 把每个值的「上一次出现位置」计入，主席树按右端点建版本，查询变成区间计数。

**树上路径第 $k$ 小.** 版本沿根路径建，路径计数 = $\mathrm{root}_u+\mathrm{root}_v-\mathrm{root}_{\mathrm{lca}}-\mathrm{root}_{\mathrm{fa}(\mathrm{lca})}$，是四棵树的线性组合（广义可减性）。

**可持久化数组 / 并查集.** 同一套路径复制，把「加计数」换成「改值」；并查集的可持久化需要按秩合并并复制父指针。

## 10 参考

- OI Wiki：[主席树（可持久化线段树）](https://oi-wiki.org/ds/persistent-seg/)
- 模板题：[P3834 【模板】可持久化线段树 2](https://www.luogu.com.cn/problem/P3834)、[P2617 Dynamic Rankings](https://www.luogu.com.cn/problem/P2617)、[P2633 Count on a tree](https://www.luogu.com.cn/problem/P2633)
- 《算法导论》第 14 章（数据结构的增广与不变量论证）

<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
