---
title: 线段树（区间乘、区间加、区间赋值）
date: 2026-10-27 23:43:00
categories:
  - 模板
  - 线段树
tags:
  - 线段树
  - 懒标记
  - 仿射变换
math: true
---

## 摘要

上一篇里「区间加」与「区间赋值」可以合并成仿射映射 $x \mapsto \alpha x + \beta$，其中 $\alpha \in \{0,1\}$。本文把 $\alpha$ 放开到任意整数，于是**区间乘**也进来了：三种操作 $x\mapsto x+v$、$x\mapsto vx$、$x\mapsto v$ 生成的变换族恰是全体整系数仿射映射

$$
\mathcal{A} = \{\, f(x) = \alpha x + \beta \;|\; \alpha, \beta \in \mathbb{Z} \,\},
$$

而 $\mathcal{A}$ 对复合封闭（引理 1）。这就是「三个懒标记可以压成两个」的全部理由。剩下的工作只有一件：把复合规则推对，尤其是**复合顺序**——顺序错了，单看每个 `apply_*` 都对，结果全错。

## 1 问题与记号

维护序列 $a_1..a_n$ 上的四种操作：

- $\mathrm{Add}(l,r,v)$：$a_i \leftarrow a_i + v$
- $\mathrm{Mul}(l,r,v)$：$a_i \leftarrow v \cdot a_i$
- $\mathrm{Assign}(l,r,v)$：$a_i \leftarrow v$
- $\mathrm{Query}(l,r)=\sum_{i=l}^r a_i$

之所以能把三者统一，是因为它们都是**对单个元素的一次仿射变换**，因此可以在结点上沉积。

## 2 变换族与封闭性

定义 $\mathrm{Aff}(\alpha,\beta): x \mapsto \alpha x + \beta$。于是

- $\mathrm{Add}(v) = \mathrm{Aff}(1, v)$
- $\mathrm{Mul}(v) = \mathrm{Aff}(v, 0)$
- $\mathrm{Assign}(v) = \mathrm{Aff}(0, v)$

**引理 1（复合封闭）.** $\mathrm{Aff}(\alpha_2,\beta_2) \circ \mathrm{Aff}(\alpha_1,\beta_1) = \mathrm{Aff}(\alpha_2\alpha_1,\; \alpha_2\beta_1+\beta_2)$。

*证明.* 直接代入：$\alpha_2(\alpha_1 x+\beta_1)+\beta_2 = (\alpha_2\alpha_1)x + (\alpha_2\beta_1+\beta_2)$。$\square$

**推论 2.** 任意有限长的操作序列都等价于一个 $\mathrm{Aff}$，因此结点上只需存一对 $(\alpha,\beta)$，与操作次数无关。

**注意**：$\mathcal{A}$ 对复合构成**半群**（有结合律、有单位元 $\mathrm{Aff}(1,0)$），但**不是群**——$\mathrm{Aff}(0,v)$（赋值）不可逆。这一点有两处后果：

1. 复合方向不能随便交换，$\mathrm{Aff}(0,v)\circ\mathrm{Aff}(u,0) \ne \mathrm{Aff}(u,0)\circ\mathrm{Aff}(0,v)$；
2. 一旦出现赋值，之前所有的乘加历史都被抹平，可以被「吸收」进一个常数。

## 3 标记的规范形

代码里存了四个字段：`is_assign`、`lazy_assign`、`lazy_mul`、`lazy_add`。它表示的映射是

$$
f_{u}(x) \;=\; m_u \cdot \bigl(\,\text{is\_assign}_u \;?\; s_u : x\,\bigr) \;+\; d_u .
$$

也就是说，语义顺序固定为：**先看是否有赋值把它变成常数，再乘 $m$，最后加 $d$**。

**引理 3（规范形可维持）.** 若 $\text{is\_assign}=1$ 时强制 $m=1, d=0$，则引理 1 的三种复合结果仍落在规范形内。

*证明.* 分情况看三种 `apply`：

- 赋值 $\mathrm{Aff}(0,v)$ 复合在左：$0\cdot(\cdots)+v \Rightarrow (s,d,m) := (v,0,1)$，进入赋值分支，规范；
- 乘 $v$：作用于 $f$ 得 $v\cdot f$。若原为赋值分支，$v\cdot s$ 仍是常数，规范；否则 $(m,d):=(vm, vd)$；
- 加 $v$：作用于 $f$ 得 $f+v$。若原为赋值分支，$s+v$ 仍是常数，规范；否则 $d := d+v$。

三种情况都保持「赋值时 $m=1,d=0$」或「非赋值时 $s$ 无意义」。$\square$

这就是代码里 `apply_add` 里那个看起来多余的 `if (is_assign) lazy_assign += val; else lazy_add += val;` —— 它不是优化，是规范形的维护。同理 `apply_mul` 里赋值时只乘 `lazy_assign`。

## 4 复合顺序：最容易错的地方

设结点上已有标记 $f$，现在要施加新操作 $g$。**新操作在时间上更晚**，所以新的合成映射是 $g \circ f$（先 $f$ 后 $g$）。把 $g$ 按上面的顺序逐层「套」到 $(s,m,d)$ 上：

1. $g=\mathrm{Assign}(v)$：$g\circ f$ 是常函数 $v$，直接重置三元组为 $(v,0,1)$；
2. $g=\mathrm{Mul}(v)$：$v\cdot\bigl(m(\cdots)+d\bigr) = (vm)(\cdots) + vd$，即 $m \leftarrow vm,\; d\leftarrow vd$；
3. $g=\mathrm{Add}(v)$：$m(\cdots)+d+v$，即 $d \leftarrow d+v$。

**引理 4（下传顺序）.** `push_down` 必须先下传赋值、再下传乘、最后下传加，才能与上面的语义顺序一致。

*证明.* 若顺序反过来（先加后乘），施加到儿子上的映射是 $x \mapsto (x+d)m$，而正确语义是 $x\mapsto mx+d$，二者相差 $dm-d$，仅在 $d=0$ 或 $m=1$ 时偶然相等。$\square$

这条引理的实用版本就是一句口诀：**写进标记的顺序是「先赋值、后乘、再加」，读出来的顺序也必须如此**。

## 5 不变量与正确性

**不变量 I.** 对每个结点 $u$：$\mathrm{dat}(u)$ 是把标记 $f_u$ 作用到 $T(u)$ 全部元素后得到的区间和；子树内更细的 $\mathrm{dat}$ 不含 $f_u$。

由 $f$ 的形状立刻得到 $\mathrm{dat}$ 的更新公式——这也是唯一需要会算的东西：

$$
\mathrm{dat}'(u) =
\begin{cases}
s_u \cdot |T(u)|, & \text{is\_assign},\\[2pt]
m_u \cdot \mathrm{dat}(u) + d_u\cdot |T(u)|, & \text{otherwise}.
\end{cases}
$$

代码里对应 `tr[rt].dat = val * len;`、`tr[rt].dat *= val;`、`tr[rt].dat += val * len;`。

**引理 5.** 按引理 4 的顺序 `push_down` 之后，不变量 I 在两个儿子上成立，且父亲的 $\mathrm{dat}$ 不变（因为 $\mathrm{dat}$ 已计入 $f$）。

**定理（正确性）.** 每次操作后 $\mathrm{dat}(\text{根})$ 等于真实区间和。

*证明提纲.* 与单标记情形同构：标准分解上的完全覆盖结点按引理 3 打标记（不变量 I 条件 1）；路径结点由引理 5 与合并 $\mathrm{dat}(u)=\mathrm{dat}(u_L)+\mathrm{dat}(u_R)$ 修复；分解外结点不受影响。$\square$

## 6 复杂度

由引理 4（标准分解至多 $2\log_2 n$ 个结点），单次操作 $O(\log n)$；每次操作常数次仿射复合。总时间 $O((n+m)\log n)$，空间 $O(m\log n)$（动态开点）。

需要注意常量本身变大了：一次 `push_down` 最多做三次 `apply`，每次都要取模。模运算下这不是零成本，$10^5$ 次操作时建议把 `int` 换成 `long long` 以省掉中途溢出判断。

## 7 实现

```cpp
class SegmentTree {
	struct node {
		int lc = -1, rc = -1;
		int dat = 0;
		int lazy_add = 0;
		int lazy_mul = 1;
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
		int len = r - l + 1;
		tr[rt].is_assign = true;
		tr[rt].lazy_assign = val;
		tr[rt].lazy_add = 0;
		tr[rt].lazy_mul = 1;
		tr[rt].dat = val * len;
		return rt;
	}

	inline int apply_add(int rt, int l, int r, int val) {
		if (rt == -1) rt = create_node();
		int len = r - l + 1;
		if (tr[rt].is_assign) tr[rt].lazy_assign += val;
		else tr[rt].lazy_add += val;
		tr[rt].dat += val * len;
		return rt;
	}

	inline int apply_mul(int rt, int l, int r, int val) {
		if (rt == -1) rt = create_node();
		if (tr[rt].is_assign) tr[rt].lazy_assign *= val;
		else {
			tr[rt].lazy_mul *= val;
			tr[rt].lazy_add *= val;
		}
		tr[rt].dat *= val;
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
		if (tr[rt].lazy_mul != 1) {
			tr[rt].lc = apply_mul(tr[rt].lc, l, mid, tr[rt].lazy_mul);
			tr[rt].rc = apply_mul(tr[rt].rc, mid + 1, r, tr[rt].lazy_mul);
			tr[rt].lazy_mul = 1;
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
		if (lb <= l && r <= ub) return apply_assign(rt, l, r, val);
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = assign(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub) tr[rt].rc = assign(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt);
		return rt;
	}

	int add(int rt, int l, int r, int lb, int ub, int val) {
		if (rt == -1) rt = create_node();
		if (lb <= l && r <= ub) return apply_add(rt, l, r, val);
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = add(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub) tr[rt].rc = add(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt);
		return rt;
	}

	int mul(int rt, int l, int r, int lb, int ub, int val) {
		if (rt == -1) rt = create_node();
		if (lb <= l && r <= ub) return apply_mul(rt, l, r, val);
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = mul(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub) tr[rt].rc = mul(tr[rt].rc, mid + 1, r, lb, ub, val);
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
		root = build(root, rangeL, rangeR, arr);
	}

	inline void assign(int pos, int val) {
		assert(rangeL <= rangeR);
		root = assign(root, rangeL, rangeR, pos, pos, val);
	}

	inline void assign(int l, int r, int val) {
		assert(rangeL <= rangeR);
		root = assign(root, rangeL, rangeR, l, r, val);
	}

	inline void add(int pos, int val) {
		assert(rangeL <= rangeR);
		root = add(root, rangeL, rangeR, pos, pos, val);
	}

	inline void add(int l, int r, int val) {
		assert(rangeL <= rangeR);
		root = add(root, rangeL, rangeR, l, r, val);
	}

	inline void mul(int pos, int val) {
		assert(rangeL <= rangeR);
		root = mul(root, rangeL, rangeR, pos, pos, val);
	}

	inline void mul(int l, int r, int val) {
		assert(rangeL <= rangeR);
		root = mul(root, rangeL, rangeR, l, r, val);
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

## 8 使用

```cpp
int n;
cin >> n;
vector<int> a(n + 1);
for (int i = 1; i <= n; i++) cin >> a[i];

SegmentTree seg(n);
seg.build(a, 1, n);

seg.mul(2, 5, 3);      // [2,5] 每个数 *3
seg.add(3, 3, 7);      // [3,3] +7
seg.assign(1, n, 0);   // 整段赋值 0，之前的标记全部作废

cout << seg.query(1, n) << "\n";
```

## 9 实现细节与易错点

- **顺序**：`push_down` 里赋值 → 乘 → 加，一步都不能换位（引理 4）。
- **赋值要重置乘加**：`apply_assign` 把 `lazy_mul` 置 1、`lazy_add` 置 0，否则残留的乘加会在之后被重复施加。
- **赋值分支不能让 `lazy_add` 累积**：`apply_add` 在 `is_assign` 时改的是 `lazy_assign`，改 `lazy_add` 会让那个 0 变成非 0，破坏规范形（引理 3）。
- **取模时机**：模 $P$ 时 `dat`、`lazy_mul`、`lazy_add`、`lazy_assign` 都要在写入时取模；尤其 `lazy_assign = val % P` 必须做，输入可能是负数。
- **乘法溢出**：$m\cdot \mathrm{dat}$ 在 $P \sim 10^9$ 时可达 $10^{18}$，用 `long long` 存中间值；本实现的字段是 `int`，模数大时要改。
- **查询也要 push_down**：查询只读，但它需要的 $\mathrm{dat}$ 在更细一层的结点上，不下传就不对。
- **`assert(rangeL <= rangeR)`**：忘了 `build` 或 `reset` 就调用会立刻暴露，而不是静默返回错误答案。
- **不要用「先乘后加」的另一套约定混搭**：另一种常见实现把标记写成 $x\mapsto (x+d)m$（先加后乘），语义顺序相反。两套都自洽，但**必须全套统一**，混用后果与引理 4 的反例相同。

## 10 扩展

**标记永久化.** 不下传，查询时沿路径累积 $f$。因为 $\mathcal{A}$ 是半群（可结合）但不可交换，累积必须按「从上到下」的顺序乘，实现上比下传更需要小心。

**区间赋值 + 区间最值.** $\mathrm{dat}$ 换成 $\max$，赋值时 $\mathrm{dat}\leftarrow v$，乘加时按 $v$ 的符号讨论（乘负数会把 $\max$ 变 $\min$），这时要同时维护 $\max$ 与 $\min$。

**可持久化.** 与单标记情形一样做路径复制；因为标记是 $(\alpha,\beta)$ 对，复制时只多存两个数。

**Beats 类结构.** 若把「区间取 min」也塞进来，操作族不再是仿射的（$\min(x,v)$ 不是仿射映射），懒标记的封闭性失效，必须转向 Segment Tree Beats 的势能分析。这是本文框架的边界。

## 11 参考

- OI Wiki：[线段树](https://oi-wiki.org/ds/seg/)
- 模板题：[P3373 【模板】线段树 2](https://www.luogu.com.cn/problem/P3373)（区间加、区间乘）、[P2023 [AHOI2009] 维护序列](https://www.luogu.com.cn/problem/P2023)
- AtCoder Library：[lazy_segtree](https://atcoder.github.io/ac-library/production/document_zh/ds/lazy_segtree.html)——它的 $F$ 半群正是把本文的 $\mathcal{A}$ 抽象出来
- 《算法导论》第 14 章：结点扩张与不变量证明的方法论

<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
