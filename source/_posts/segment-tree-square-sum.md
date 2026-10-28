---
title: 线段树（区间加、区间乘，维护和与平方和）
date: 2026-10-27 23:44:00
categories:
  - 模板
  - 线段树
tags:
  - 线段树
  - 懒标记
  - 平方和
  - 二级矩
math: true
---

## 摘要

维护区间和只需在结点上存一个数；维护**区间平方和**时，一次「整段加 v」会通过多项式展开同时牵动和与平方和：

$$\sum_{i\in T}(x_i+v)^2 = \sum_{i\in T}x_i^2 + 2v\sum_{i\in T}x_i + v^2|T|.$$

于是必须同时维护 $\sum x_i$ 与 $\sum x_i^2$，并且在施加标记时保持**更新顺序**正确——先更新平方和、后更新和。本文把这件事说成「二级矩的仿射变换公式」，给出闭包性引理、正确性不变量，以及模板常数与溢出边界。

## 1 问题与记号

维护序列在 $\mathrm{Add}(l,r,v)$、$\mathrm{Mul}(l,r,v)$、$\mathrm{Assign}(l,r,v)$ 三种操作下的

$$\mathrm{Sum}(l,r)=\sum_{i=l}^rx_i, \qquad \mathrm{Sq}(l,r)=\sum_{i=l}^rx_i^2.$$

对结点 $u$ 记 $T=T(u)$、$L=|T|$，并记

$$S_1(T)=\sum_{i\in T}x_i,\qquad S_2(T)=\sum_{i\in T}x_i^2.$$

$S_1,S_2$ 是分布的一阶与二阶矩（未归一化）。**两者对不交并都可加**：$S_k(T_L\cup T_R)=S_k(T_L)+S_k(T_R)$，$k=1,2$。这是 push_up 合法的全部理由，也是「为什么维护矩而不是直接维护答案」的答案。

## 2 仿射变换对矩的作用

设对区间内每个元素施加一次仿射映射 $f(x)=mx+d$，得到新元素 $y_i=mx_i+d$。

**引理 1（一阶矩）.** $S_1' = mS_1 + dL$。

**引理 2（二阶矩）.** $S_2' = m^2S_2 + 2md\,S_1 + d^2L$。

*证明.* 把 $y_i$ 代入并按 $x_i$ 的幂次归并：

$$\sum_i(mx_i+d)^2=\sum_i\left(m^2x_i^2+2md\,x_i+d^2\right)=m^2S_2+2md\,S_1+d^2L.$$

$\square$

**推论 3（为什么要同时存两个量）.** $S_2'$ 依赖 $S_1$，所以只维护 $S_2$ 无法在 $d\neq 0$ 时更新；而 $S_1'$ 不依赖 $S_2$。因此结点上的 $(S_1,S_2)$ 是**不可分割的一对状态**。

**推论 4（赋值是特例）.** 赋值即 $m=0,d=v$：$S_1'=vL$、$S_2'=v^2L$，与引理 1、2 一致。实现里直接覆盖，省两次乘法。

## 3 更新顺序引理

代码里 `apply_add` 的这一段是本文唯一「顺序敏感」的位置：

```cpp
tr[rt].dat.sq += 2 * val * tr[rt].dat.sum + val * val * len;
tr[rt].dat.sum += val * len;
```

**引理 5.** 若先更新 `sum` 再更新 `sq`，结果错误。

*证明.* 引理 2 里的 $S_1$ 必须是**变换前**的一阶矩。若先把 `sum` 改成 $S_1+dL$，则 `sq` 会算成

$$S_2+2d(S_1+dL)+d^2L=S_2+2dS_1+3d^2L,$$

比正确值多出 $d^2L$，且误差在每次下传时累积。$\square$

**推论 6.** `apply_*` 中凡「先读后改」的量（这里是 `sum`），必须把所有读取放完再写。`apply_assign` 与 `apply_mul` 的更新互相独立，顺序无所谓——**只有加法这一支有顺序要求**，而它恰好最容易写反。

这个错误还有一个隐蔽之处：当 $d=0$（加 0）或被加的数恰好为 0 时，两种写法结果相同，随机对拍很容易漏掉。

## 4 标记与不变量

标记仍是三元组（赋值常数 $s$、乘 $m$、加 $d$），语义顺序「先赋值、再乘、后加」：

$$f_u(x)=\text{is\_assign}_u\;?\;m_us_u+d_u\;:\;m_ux+d_u,$$

规范形：赋值时 $m=1,d=0$。复合规则与上一篇完全相同（赋值覆盖、乘同时乘 $m,d$、加只加 $d$）。

**不变量 I.** $\mathrm{dat}(u)=(S_1,S_2)$ 等于把 $f_u$ 作用到 $T(u)$ 每个元素后的一阶、二阶矩；子树更细的结点不含 $f_u$。

**引理 7.** 按引理 1、2、5 更新 $\mathrm{dat}$ 并维护规范形，则任意复合后不变量 I 成立。

*证明.* 三种 `apply` 分别是 $g\circ f$，其中 $g$ 取赋值/乘/加；由引理 1、2 得新的 $(S_1,S_2)$；规范形由上一篇的引理 3 保证。$\square$

**定理.** 每次操作后根结点的 $\mathrm{dat}$ 等于真实答案。证明与上一篇同构（标准分解、引理 7、可加性）。

## 5 复杂度

标准分解至多 $2\log_2 n$ 个结点，单次操作 $O(\log n)$。结点常数工作约 4 次乘法（$m\cdot m$、$2md$、$d^2L$、$mS_1$），比只维护和的版本重一档。空间 $O(m\log n)$。

## 6 实现

本实现**不带取模**，全部 `long long`，适合 $|x_i|,|v|\le 10^4$、$n\le 10^5$ 的场景（此时 $|S_2|\lesssim 10^{14}$，乘法中间量仍在 `long long` 内）。

```cpp
class SegmentTree {
	struct DATA {
		long long sum = 0;
		long long sq = 0;
	};

	struct node {
		int lc = -1, rc = -1;
		DATA dat;
		long long lazy_add = 0;
		long long lazy_mul = 1;
		long long lazy_assign = 0;
		bool is_assign = false;
	};

	vector<node> tr;
	int root = 0, rangeL, rangeR;

	inline int create_node() {
		tr.emplace_back();
		return tr.size() - 1;
	}

	inline int apply_assign(int rt, int l, int r, long long val) {
		if (rt == -1) rt = create_node();
		long long len = r - l + 1;
		tr[rt].is_assign = true;
		tr[rt].lazy_assign = val;
		tr[rt].lazy_add = 0;
		tr[rt].lazy_mul = 1;
		tr[rt].dat.sum = val * len;
		tr[rt].dat.sq = val * val * len;
		return rt;
	}

	inline int apply_add(int rt, int l, int r, long long val) {
		if (rt == -1) rt = create_node();
		long long len = r - l + 1;
		tr[rt].dat.sq += 2 * val * tr[rt].dat.sum + val * val * len;
		tr[rt].dat.sum += val * len;
		if (tr[rt].is_assign) tr[rt].lazy_assign += val;
		else tr[rt].lazy_add += val;
		return rt;
	}

	inline int apply_mul(int rt, int l, int r, long long val) {
		if (rt == -1) rt = create_node();
		tr[rt].dat.sum *= val;
		tr[rt].dat.sq *= val * val;
		if (tr[rt].is_assign) tr[rt].lazy_assign *= val;
		else {
			tr[rt].lazy_mul *= val;
			tr[rt].lazy_add *= val;
		}
		return rt;
	}

	inline void push_up(int rt) {
		int lc = tr[rt].lc, rc = tr[rt].rc;
		tr[rt].dat.sum = (lc != -1 ? tr[lc].dat.sum : 0) + (rc != -1 ? tr[rc].dat.sum : 0);
		tr[rt].dat.sq = (lc != -1 ? tr[lc].dat.sq : 0) + (rc != -1 ? tr[rc].dat.sq : 0);
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

	int build(int rt, int l, int r, const vector<long long> &arr) {
		if (rt == -1) rt = create_node();
		if (l == r) {
			tr[rt].dat.sum = arr[l];
			tr[rt].dat.sq = arr[l] * arr[l];
			return rt;
		}
		int mid = (l + r) >> 1;
		tr[rt].lc = build(tr[rt].lc, l, mid, arr);
		tr[rt].rc = build(tr[rt].rc, mid + 1, r, arr);
		push_up(rt);
		return rt;
	}

	int assign(int rt, int l, int r, int lb, int ub, long long val) {
		if (rt == -1) rt = create_node();
		if (lb <= l && r <= ub) return apply_assign(rt, l, r, val);
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = assign(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub) tr[rt].rc = assign(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt);
		return rt;
	}

	int add(int rt, int l, int r, int lb, int ub, long long val) {
		if (rt == -1) rt = create_node();
		if (lb <= l && r <= ub) return apply_add(rt, l, r, val);
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = add(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub) tr[rt].rc = add(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt);
		return rt;
	}

	int mul(int rt, int l, int r, int lb, int ub, long long val) {
		if (rt == -1) rt = create_node();
		if (lb <= l && r <= ub) return apply_mul(rt, l, r, val);
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = mul(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub) tr[rt].rc = mul(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt);
		return rt;
	}

	DATA query(int rt, int l, int r, int lb, int ub) {
		if (rt == -1) return DATA{};
		if (lb <= l && r <= ub) return tr[rt].dat;
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		DATA res{};
		if (lb <= mid) {
			DATA tmp = query(tr[rt].lc, l, mid, lb, ub);
			res.sum += tmp.sum;
			res.sq += tmp.sq;
		}
		if (mid < ub) {
			DATA tmp = query(tr[rt].rc, mid + 1, r, lb, ub);
			res.sum += tmp.sum;
			res.sq += tmp.sq;
		}
		return res;
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

	inline void build(const vector<long long> &arr, int l, int r) {
		reset(l, r);
		root = build(root, rangeL, rangeR, arr);
	}

	inline void assign(int pos, long long val) {
		assert(rangeL <= rangeR);
		root = assign(root, rangeL, rangeR, pos, pos, val);
	}

	inline void assign(int l, int r, long long val) {
		assert(rangeL <= rangeR);
		root = assign(root, rangeL, rangeR, l, r, val);
	}

	inline void add(int pos, long long val) {
		assert(rangeL <= rangeR);
		root = add(root, rangeL, rangeR, pos, pos, val);
	}

	inline void add(int l, int r, long long val) {
		assert(rangeL <= rangeR);
		root = add(root, rangeL, rangeR, l, r, val);
	}

	inline void mul(int pos, long long val) {
		assert(rangeL <= rangeR);
		root = mul(root, rangeL, rangeR, pos, pos, val);
	}

	inline void mul(int l, int r, long long val) {
		assert(rangeL <= rangeR);
		root = mul(root, rangeL, rangeR, l, r, val);
	}

	inline long long query_sum(int pos) {
		assert(rangeL <= rangeR);
		return query(root, rangeL, rangeR, pos, pos).sum;
	}

	inline long long query_sum(int l, int r) {
		assert(rangeL <= rangeR);
		return query(root, rangeL, rangeR, l, r).sum;
	}

	inline long long query_sq(int pos) {
		assert(rangeL <= rangeR);
		return query(root, rangeL, rangeR, pos, pos).sq;
	}

	inline long long query_sq(int l, int r) {
		assert(rangeL <= rangeR);
		return query(root, rangeL, rangeR, l, r).sq;
	}
};
```

## 7 使用

```cpp
int n;
cin >> n;
vector<long long> a(n + 1);
for (int i = 1; i <= n; i++) cin >> a[i];

SegmentTree seg(n);
seg.build(a, 1, n);

seg.add(2, 5, 3);        // [2,5] 每个数 +3
seg.mul(1, n, 2);        // 整段 *2
seg.assign(4, 4, 10);    // 单点赋值

cout << seg.query_sum(1, n) << "\n";   // 区间和
cout << seg.query_sq(1, n) << "\n";    // 区间平方和
```

## 8 实现细节与易错点

- **先 sq 后 sum**（引理 5）：写反了在 $d=0$ 的样例上测不出来，必须刻意用非零加数测。
- **溢出**：$2mdS_1$ 的量级是 $|m||d|n|x|^2$。本模板在 $10^4$ 级数据上刚好安全，数据一大就要换 `__int128` 或取模。
- **取模版本**：$m^2$、$d^2$ 先取模再乘；$2mdS_1$ 用 `long long` 算完再取模；$d<0$ 时记得 $+P$ 归一。
- **赋值要同时重置三个字段**：`dat.sum = val*len`、`dat.sq = val*val*len`、`lazy_mul=1, lazy_add=0`。
- **建树时 sq 的初值是 $x_l^2$**：不是 $x_l$；push_up 里两个矩都要加，不能只加 sum。
- **合并查询**：`query_sum` 与 `query_sq` 各自 $O(\log n)$；若要一次拿两个值，让查询返回 `DATA`，可以省一次下传路径。
- **负数与溢出混淆**：平方和天然非负，算出负数只可能是溢出或取模没归一，别怀疑公式。

## 9 扩展

**更高阶矩.** 施加仿射变换后，$k$ 阶矩需要 $0..k$ 阶矩才能更新（二项式展开），每结点代价 $O(k^2)$。$k=2$ 是最常见的实用情形。

**方差.** $\mathrm{Var}=\frac{S_2}{L}-\left(\frac{S_1}{L}\right)^2$。方差对合并不封闭（要除以长度），矩可加——这就是维护矩而不是维护方差的原因。

**加权平方和.** 若每个位置有权 $w_i$，维护 $\sum w_ix_i$ 与 $\sum w_ix_i^2$，引理 1、2 中 $L$ 换成 $W=\sum w_i$ 即可，是「带权矩」。

**位运算版.** 若把 $+$ 换成 $\oplus$，各二进制位独立，可按位维护「1 的个数」，是另一套技巧（按位线段树），与本框架正交。

## 10 参考

- OI Wiki：[线段树](https://oi-wiki.org/ds/seg/)
- 模板题：[P1471 方差](https://www.luogu.com.cn/problem/P1471)（维护和与平方和求区间方差）、[P3373 【模板】线段树 2](https://www.luogu.com.cn/problem/P3373) 的矩版变体
- 《算法导论》第 14 章：结点扩张与不变量证明的方法论

<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
