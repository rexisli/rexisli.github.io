---
title: 线段树（区间加、区间赋值）
date: 2026-10-27 23:40:00
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

## 怎么用

```cpp
int n;
cin >> n;
vector<int> a(n + 1);
for (int i = 1; i <= n; i++) cin >> a[i];

SegmentTree seg(n);          // 维护区间 [1, n]；也可以 SegmentTree seg(l, r) 指定任意下标区间
seg.build(a, 1, n);          // arr 的下标就是线段树下标，build(a, l, r) 表示用 a[l..r] 建树

seg.add(3, 5, 10);           // [3,5] 每个数 +10
seg.assign(2, 7, 0);         // [2,7] 全部赋值为 0
seg.add(4, -1);              // 单点，等价于 add(4, 4, -1)
seg.assign(1, 100);          // 单点赋值

cout << seg.query(1, n) << "\n";   // 区间和
cout << seg.query(4) << "\n";      // 单点查询

seg.reset(1, m);             // 换一个区间重用（会清空）
```

## 说明

这是一棵**动态开点**线段树：一开始只有根，访问到哪段才 `create_node()`，所以下标范围可以开到 $1 \le n \le 10^9$，只要操作次数 $m$ 满足 $m \log n$ 个点就够。

区间赋值和区间加是两个会互相覆盖的标记：

- `assign(l, r, v)` 打上 `is_assign = true`、`lazy_assign = v`，同时把 `lazy_add` 清零——赋值之后，之前的加法已经没有意义了。
- `add(l, r, v)` 时如果这一段已经被整体赋值过，就**直接改赋值标记**（`lazy_assign += v`），不再压一个加法标记：一段区间先整体赋值再加常数，本质还是整体赋值成另一个常数。
- `push_down` 的顺序是「先下传赋值，再下传加法」。

## 易错点

- `push_down` 之后递归更新孩子，最后一定 `push_up`，否则父亲结点会一直停在旧值。
- 查询前要 `push_down`，否则孩子结点的 `dat` 是旧的。
- 下标体系是**闭区间**，判重叠写 `if (lb <= mid)` / `if (mid < ub)`，不是 `l <= mid`。
- 空子树用 `-1` 表示，`push_up` 里按 0 处理即可。
- `assert(rangeL <= rangeR)` 是在提醒你：建树之前直接查询会直接 RE，`reset` 之后才是合法状态。
<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
