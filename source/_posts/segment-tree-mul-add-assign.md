---
title: 线段树（区间乘、区间加、区间赋值）
date: 2026-10-27 23:43:00
categories:
  - 模板
  - 线段树
tags:
  - 线段树
  - 懒标记
---

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

## 怎么用

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

## 说明

三个标记的优先级是 **赋值 > 乘法 > 加法**，下传顺序也固定为 `assign → mul → add`。

- 来了一个乘法，结点上挂着的加法也要跟着乘（$(x+a)\times v = xv+av$），所以 `lazy_add *= val`；
- 来了一个加法，结点上挂着的乘法**不用动**，因为乘法在加法之前生效；
- 来了一个赋值，之前的一切都被覆盖：`lazy_add` 清 0、`lazy_mul` 归一。

`is_assign` 为真时，这一段的值恒等于 `lazy_assign`，此时乘法和加法都只改 `lazy_assign` 本身，不用再维护另外两个标记。

## 易错点

- `push_down` 里三个分支的顺序不能换，换掉就是另一个语义。
- 取模的题要在 `dat`、`lazy_mul`、`lazy_add` 的每次赋值处取模；这个模板是纯整数版本，没有模数。
- 乘法标记初值是 1 不是 0，构造函数和 `clear()` 里都要盯住。
- 区间赋值会把历史标记全部作废——这正是「赋值 > 乘法 > 加法」能够自洽的原因。

<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
