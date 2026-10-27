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
---

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

## 怎么用

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

## 说明

维护平方和只多一条关键规则：**先更新平方和，再更新和**。

$$\sum (x_i + a)^2 = \sum x_i^2 + 2a\sum x_i + a^2 \cdot len$$

公式里用到的 $\sum x_i$ 是「加之前」的和，所以 `apply_add` 里必须先算 `sq` 再算 `sum`；写反了就会用到已经被加过的和，答案偏大。

乘法的公式是

$$\sum (x_i \cdot v)^2 = v^2 \sum x_i^2$$

所以 `sq *= v * v`、`sum *= v`。

## 易错点

- 这里刻意用了 `long long`：平方和极易爆 `int`，$10^5$ 个 $10^9$ 的平方已经到 $10^{23}$ 量级，读题时先看取值范围再决定类型。
- `apply_add` 里 `sq`、`sum` 两行的顺序不能换。
- 查询返回 `DATA{}`（全 0）表示空子树，不用额外特判。
- 有模数时：$2a\sum x_i$ 里的 $2$、$a^2$ 和 $len$ 都要取模。

<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
