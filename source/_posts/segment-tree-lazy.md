---
title: 线段树（区间加、区间赋值）
date: 2026-10-27 23:40:00
categories:
  - 模板
  - 线段树
tags:
  - 线段树
  - 懒标记
math: true
---

## 摘要

线段树用一棵二叉树把序列的每个区间「分解」成 $O(\log n)$ 个标准区间，从而让区间修改与区间查询都落在 $O(\log n)$ 个结点上。当修改不是单点而是**整段**时，我们不再逐个下探到叶子，而是把「这一整段都要加 $v$」这件事记录在结点上——这就是懒标记。本文讨论两种同时存在、彼此覆盖的整段修改：**区间赋值**与**区间加**，给出标记的代数描述、正确性不变量、$O(\log n)$ 的复杂度证明，以及一份可直接使用的动态开点实现。

## 1. 问题与记号

给定长度为 $n$ 的序列 $a_1,\dots,a_n$（下标从 $1$ 开始），维护两种修改与一种查询：

- $\mathrm{Add}(l,r,v)$：对所有 $l \le i \le r$，令 $a_i \leftarrow a_i + v$；
- $\mathrm{Assign}(l,r,v)$：对所有 $l \le i \le r$，令 $a_i \leftarrow v$；
- $\mathrm{Query}(l,r) = \sum_{i=l}^{r} a_i$。

朴素做法每次操作 $O(n)$，$m$ 次操作 $O(nm)$。线段树把它压到 $O((n+m)\log n)$。

记 $T(u)$ 为结点 $u$ 对应的区间，$|T(u)|$ 为其长度，$\mathrm{dat}(u)$ 为该区间元素之和。若 $T(u)$ 的左右儿子分别是 $u_L, u_R$，则 $T(u) = T(u_L) \cup T(u_R)$ 且两者不交。

## 2. 核心思想

序列的区间可以递归二分，于是任意区间 $[l,r]$ 都可以被唯一地**标准分解**为若干个满足「要么完全包含、要么完全不相交」的结点区间之并。只在这些结点上打标记，就等价于对 $[l,r]$ 做了一次修改——**前提是我们能保证将来需要访问更细粒度时会「补上」这些标记**。

标记因此需要两件事：

1. **语义**：一个结点上挂的标记，表示「对这整段的每个元素还要施加的待办操作」；
2. **复合**：多个待办操作要能合成一个等价的操作，否则标记会越积越多。

## 3. 标记的代数描述

设结点 $u$ 上挂了标记 $(c, s, d)$，其中 $c$ 表示「是否待办了一次赋值」、$s$ 是待赋值、$d$ 是待加的增量。定义它对单个元素的**作用函数**

$$
f_u(x) \;=\;
\begin{cases}
s_u + d_u, & c_u = 1,\\[2pt]
x + d_u,   & c_u = 0.
\end{cases}
$$

也就是说：先（可能）把 $x$ 抹成 $s_u$，再加上 $d_u$。这是一个**仿射映射** $x \mapsto \alpha x + \beta$，只不过这里的 $\alpha$ 只能取 $0$ 或 $1$。

**引理 1（标记复合封闭）.** 两个形如 $f(x)=\alpha x+\beta$（$\alpha\in\{0,1\}$）的映射复合后仍是同一形式。

*证明.* $\alpha_2(\alpha_1 x + \beta_1) + \beta_2 = (\alpha_2\alpha_1)x + (\alpha_2\beta_1+\beta_2)$，而 $\alpha_2\alpha_1 \in \{0,1\}$。$\square$

这条引理是懒标记能成立的根源：**任意长的待办操作序列都能压缩成三元组** $(c,s,d)$。

对应到代码：

- 施加赋值 $v$：$f \leftarrow (x \mapsto v)$，即 `c=1, s=v, d=0`；
- 施加加 $v$：$f \leftarrow f(x) + v$，即 $d \leftarrow d + v$（若原本 $c=1$，由于 $s+d$ 仍是常数，语义依然正确）。

注意第二种情况**不需要**特判 $c$：当 $c=1$ 时 $s+d$ 就是新的赋值常数，二者一致。

## 4. 不变量与正确性

**不变量 I.** 对任意结点 $u$：

1. $\mathrm{dat}(u)$ 等于「把 $f_u$ 作用到 $T(u)$ 内每个元素后」的区间和；
2. 对 $u$ 的子树中每个结点 $w$，其 $\mathrm{dat}(w)$ 描述的是**尚未包含** $f_u$ 的旧值。

换言之：$u$ 的标记是「已记账、未下传」。$\mathrm{dat}$ 永远跟着标记同步更新，所以根结点的 $\mathrm{dat}$ 就是整段答案。

**引理 2（push_down 保不变量）.** 把 $f_u$ 下传给两个儿子并清空 $f_u$，不变量 I 仍然成立。

*证明.* 对儿子 $v$ 来说，新的待办是 $f_u \circ f_v$，由引理 1 可写成三元组；其 $\mathrm{dat}$ 更新为 $f_u$ 作用后的值。由 $T(u)=T(u_L)\cup T(u_R)$ 且不交，根的和不变。清空 $f_u$ 后 $u$ 变回恒等，条件 2 也成立。$\square$

**引理 3（push_up 保不变量）.** 儿子维护正确时，$\mathrm{dat}(u)=\mathrm{dat}(u_L)+\mathrm{dat}(u_R)$ 使 $u$ 重新正确。

这是因为加法对不相交集合可加，与标记无关。

**定理（正确性）.** 按「完全覆盖 → 打标记；部分相交 → 先 push_down，递归，再 push_up」执行，每次操作后 $\mathrm{dat}(\text{根})$ 等于真实答案。

*证明提纲.* 对操作区间做标准分解，完全覆盖的结点按引理 1 打标记（保持不变量 I 条件 1）；路径上的结点由引理 2 与 3 修复；分解之外的结点不被触碰，其上标记仍描述正确语义。对 $m$ 归纳即得。$\square$

## 5. 复杂度

**引理 4.** 任意区间 $[l,r]$ 在递归二分树上的标准分解至多含 $2\log_2 n$ 个结点。

*证明思路.* 分解出的结点分为「左边界链上的」与「右边界链上的」两类，每条链的深度不超过树高 $\log_2 n$；完全被覆盖的中间段在两条链之间自动闭合，不会再展开。$\square$

于是每次操作访问的结点数是 $O(\log n)$，单次操作时间 $O(\log n)$（每结点常数工作），空间 $O(n)$（静态建树）或 $O(m\log n)$（动态开点，见第 6 节）。

## 6. 实现

下面这份实现是**动态开点**版本：不预先建满 $4n$ 个结点，而是一路 `create_node()`，因此下标范围可以开到 $10^9$ 级别。

```cpp
class SegmentTree {
	struct node {
		int lc = -1, rc = -1;
		int dat = 0;
		int lazy_add = 0;
		int lazy_assign = 0;
		bool is_assign = false;
	};

	vector<node> tr;
	int root = 0, rangeL, rangeR;

	inline int create_node() {
		tr.emplace_back();
		return tr.size() - 1;
	}

	inline int apply_assign(int rt, int l, int r, int val) {
		if (rt == -1) rt = create_node();
		tr[rt].is_assign = true;
		tr[rt].lazy_assign = val;
		tr[rt].lazy_add = 0;
		tr[rt].dat = val * (r - l + 1);
		return rt;
	}

	inline int apply_add(int rt, int l, int r, int val) {
		if (rt == -1) rt = create_node();
		tr[rt].lazy_add += val;
		tr[rt].dat += val * (r - l + 1);
		return rt;
	}

	inline void push_up(int rt) {
		int res = 0;
		res += (tr[rt].lc != -1 ? tr[tr[rt].lc].dat : 0);
		res += (tr[rt].rc != -1 ? tr[tr[rt].rc].dat : 0);
		tr[rt].dat = res;
	}

	inline void push_down(int rt, int l, int r) {
		int mid = (l + r) >> 1;
		if (tr[rt].is_assign) {
			tr[rt].lc = apply_assign(tr[rt].lc, l, mid, tr[rt].lazy_assign);
			tr[rt].rc = apply_assign(tr[rt].rc, mid + 1, r, tr[rt].lazy_assign);
			tr[rt].is_assign = false;
		}
		if (tr[rt].lazy_add) {
			tr[rt].lc = apply_add(tr[rt].lc, l, mid, tr[rt].lazy_add);
			tr[rt].rc = apply_add(tr[rt].rc, mid + 1, r, tr[rt].lazy_add);
			tr[rt].lazy_add = 0;
		}
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
		if (rt == -1) rt = create_node();
		if (lb <= l && r <= ub) {
			rt = apply_assign(rt, l, r, val);
			return rt;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = assign(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub) tr[rt].rc = assign(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt);
		return rt;
	}

	int add(int rt, int l, int r, int lb, int ub, int val) {
		if (rt == -1) rt = create_node();
		if (lb <= l && r <= ub) {
			rt = apply_add(rt, l, r, val);
			return rt;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = add(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub) tr[rt].rc = add(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt);
		return rt;
	}

	int query(int rt, int l, int r, int lb, int ub) {
		if (rt == -1) return 0;
		if (lb <= l && r <= ub) return tr[rt].dat;
		push_down(rt, l, r);
		int ans = 0;
		int mid = (l + r) >> 1;
		if (lb <= mid) ans += query(tr[rt].lc, l, mid, lb, ub);
		if (mid < ub) ans += query(tr[rt].rc, mid + 1, r, lb, ub);
		return ans;
	}

public:
	SegmentTree() : rangeL(1), rangeR(0) {
		tr.emplace_back();
	}
	explicit SegmentTree(int l, int r) : rangeL(l), rangeR(r) {
		tr.emplace_back();
	}
	explicit SegmentTree(int n) : SegmentTree(1, n) {}

	inline void clear() {
		rangeL = 1, rangeR = 0;
		tr.clear(), tr.emplace_back();
	}

	inline void reset(int l, int r) {
		clear();
		rangeL = l, rangeR = r;
	}

	inline void build(const vector<int> &arr, int l, int r) {
		reset(l, r);
		build(root, rangeL, rangeR, arr);
	}

	inline void assign(int pos, int val) {
		assert(rangeL <= rangeR);
		assign(root, rangeL, rangeR, pos, pos, val);
	}

	inline void assign(int l, int r, int val) {
		assert(rangeL <= rangeR);
		assign(root, rangeL, rangeR, l, r, val);
	}

	inline void add(int pos, int val) {
		assert(rangeL <= rangeR);
		add(root, rangeL, rangeR, pos, pos, val);
	}

	inline void add(int l, int r, int val) {
		assert(rangeL <= rangeR);
		add(root, rangeL, rangeR, l, r, val);
	}

	inline int query(int pos) {
		assert(rangeL <= rangeR);
		return query(root, rangeL, rangeR, pos, pos);
	}

	inline int query(int l, int r) {
		assert(rangeL <= rangeR);
		return query(root, rangeL, rangeR, l, r);
	}
};
```

## 7. 使用

```cpp
int n;
cin >> n;
vector<int> a(n + 1);
for (int i = 1; i <= n; i++) cin >> a[i];

SegmentTree seg(n);          // 维护 [1, n]；也可 SegmentTree seg(l, r) 指定任意下标区间
seg.build(a, 1, n);          // 用 a[l..r] 建树，线段树下标与 arr 下标一致

seg.add(3, 5, 10);           // [3,5] 每个数 +10
seg.assign(2, 7, 0);         // [2,7] 整体赋值为 0
seg.add(4, -1);              // 单点加，等价于 add(4, 4, -1)
seg.assign(1, 100);          // 单点赋值

cout << seg.query(1, n) << "\n";   // 区间和
cout << seg.query(4) << "\n";      // 单点查询

seg.reset(1, m);             // 换一个区间重用（会清空所有结点）
```

## 8. 实现细节与易错点

- **push_down 的顺序**：先下传赋值、再下传加法。这不是风格问题——若先加后赋值，加法的效果会被赋值直接吃掉，结果错。
- **空儿子也要建点**：动态开点下，`push_down` 里 `apply_*` 必须能接受 `rt == -1`，否则把标记推给一个不存在的儿子就丢信息了。
- **递归回来必须 push_up**：只改孩子不回收，父亲会永远停在旧值，查询时一路错到根。
- **查询也要 push_down**：查询不改变数据，但它需要更细粒度的 `dat`，而更细的 `dat` 只有在标记下传后才正确。
- **assign 时务必把 lazy_add 清 0**：赋值抹掉了这段之前的一切加法历史，旧 `lazy_add` 若保留会被再次下传，凭空多算。
- **int 溢出**：`dat = val * (r - l + 1)` 在 $n \sim 10^5$、$|val| \sim 10^9$ 时必然爆 `int`，这种题要把 `dat`/`lazy_*` 换成 `long long`。
- **$l=r$ 时不要 push_down**：叶子没有儿子，虽然本实现里 `apply_*` 会安全建点，但那是无意义的浪费（区间赋值/加在叶子上会走「完全覆盖」分支，实际不会进 `push_down`）。

## 9. 扩展

**区间乘 + 区间加 + 区间赋值.** 把元素操作推广成仿射 $x \mapsto a x + b$，标记变成三元组 $(a,b)$，复合即 $(a_2,b_2)\circ(a_1,b_1) = (a_2a_1,\; a_2b_1+b_2)$。引理 1 在这里升级为「仿射映射对复合封闭」，写法与本文完全同构。区间赋值是 $a=0$ 的特例，所以三种操作可以统一处理，只是要小心 $a=0$ 时 `dat` 也要整体置为 $b\cdot|T|$。

**标记永久化.** 不下传标记，而是在查询路径上累积标记。省掉 push_down 的常数，但要求标记满足可交换性，而区间赋值这类会破坏交换性的操作很难处理。

**可持久化.** 修改时复制路径上的结点（路径复制），得到历史版本。复杂度 $O(\log n)$ 每版本，空间 $O((n+m)\log n)$。

**其它可合并信息.** 只要合并函数满足结合律（区间最大/最小、异或和、矩阵乘积、线性变换复合），把 $\mathrm{dat}$ 换掉即可；此时「懒标记如何作用于 $\mathrm{dat}$」需要单独推导，例如区间加作用于区间最大值时是整体平移。

## 10. 参考

- OI Wiki：[线段树](https://oi-wiki.org/ds/seg/)（含懒标记与标记永久化的对比）
- cp-algorithms: [Segment Tree](https://cp-algorithms.com/data_structures/segment_tree.html)
- AtCoder Library: [lazy_segtree](https://atcoder.github.io/ac-library/production/document_zh/ds/lazy_segtree.html)（把「标记 + 合并」抽象成 S 与 F 两个半群，本文第 3 节正是它的具体化）
- 模板题：[P3372 【模板】线段树 1](https://www.luogu.com.cn/problem/P3372)（区间加、区间和）、[P3373 【模板】线段树 2](https://www.luogu.com.cn/problem/P3373)（区间加、区间乘）
- 《算法导论》第 14 章「数据结构的扩张」：在结点上维护额外信息并证明不变量的一般方法论。

<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
