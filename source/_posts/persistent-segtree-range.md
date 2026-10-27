---
title: 主席树（区间加、区间赋值）
date: 2026-10-27 23:42:00
categories:
  - 模板
  - 线段树
tags:
  - 主席树
  - 可持久化线段树
  - 懒标记
---

```cpp
class PresidentSegmentTree {
	struct node {
		int lc = -1, rc = -1;
		int dat = 0;
		int lazy_add = 0;
		int lazy_assign = 0;
		bool is_assign = false;
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
		if (tr[rt].is_assign) {
			tr[rt].lazy_assign += val;
		} else {
			tr[rt].lazy_add += val;
		}
		tr[rt].dat += val * (r - l + 1);
		return rt;
	}

	inline void push_up(int rt, int l, int r) {
		int res = 0;
		if (tr[rt].is_assign) {
			res = (tr[rt].lazy_assign + tr[rt].lazy_add) * (r - l + 1);
		} else {
			res += (tr[rt].lc != -1 ? tr[tr[rt].lc].dat : 0);
			res += (tr[rt].rc != -1 ? tr[tr[rt].rc].dat : 0);
			res += tr[rt].lazy_add * (r - l + 1);
		}
		tr[rt].dat = res;
	}

	inline void push_down(int rt, int l, int r) {
		int mid = (l + r) >> 1;
		if (tr[rt].is_assign) {
			tr[rt].lc = clone_node(tr[rt].lc);
			tr[rt].rc = clone_node(tr[rt].rc);
			apply_assign(tr[rt].lc, l, mid, tr[rt].lazy_assign);
			apply_assign(tr[rt].rc, mid + 1, r, tr[rt].lazy_assign);
			tr[rt].is_assign = false;
		}
		if (tr[rt].lazy_add) {
			tr[rt].lc = clone_node(tr[rt].lc);
			tr[rt].rc = clone_node(tr[rt].rc);
			apply_add(tr[rt].lc, l, mid, tr[rt].lazy_add);
			apply_add(tr[rt].rc, mid + 1, r, tr[rt].lazy_add);
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
		push_up(rt, l, r);
		return rt;
	}

	int assign(int rt, int l, int r, int lb, int ub, int val) {
		rt = clone_node(rt);
		if (lb <= l && r <= ub) {
			rt = apply_assign(rt, l, r, val);
			return rt;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = assign(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub)  tr[rt].rc = assign(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt, l, r);
		return rt;
	}

	int add(int rt, int l, int r, int lb, int ub, int val) {
		rt = clone_node(rt);
		if (lb <= l && r <= ub) {
			rt = apply_add(rt, l, r, val);
			return rt;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (lb <= mid) tr[rt].lc = add(tr[rt].lc, l, mid, lb, ub, val);
		if (mid < ub)  tr[rt].rc = add(tr[rt].rc, mid + 1, r, lb, ub, val);
		push_up(rt, l, r);
		return rt;
	}

	int query(int rt, int l, int r, int lb, int ub) {
		if (rt == -1) return 0;
		if (lb <= l && r <= ub) return tr[rt].dat;
		int L = max(l, lb);
		int R = min(r, ub);
		int len = R - L + 1;
		if (tr[rt].is_assign) return (tr[rt].lazy_assign + tr[rt].lazy_add) * len;
		int res = 0;
		int mid = (l + r) >> 1;
		if (lb <= mid) res += query(tr[rt].lc, l, mid, lb, ub);
		if (mid < ub)  res += query(tr[rt].rc, mid + 1, r, lb, ub);
		if (tr[rt].lazy_add) res += tr[rt].lazy_add * len;
		return res;
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

	explicit PresidentSegmentTree(int n) : PresidentSegmentTree(1, n) {}

	inline void clear() {
		rangeL = 1, rangeR = 0;
		tr.clear(), tr.emplace_back();
		root.clear();
		root.emplace_back(-1);
	}

	inline void reset(int l, int r) {
		clear();
		rangeL = l, rangeR = r;
	}

	inline void build(const vector<int>& arr, int l, int r) {
		reset(l, r);
		root.back() = build(-1, rangeL, rangeR, arr);
	}

	inline void assign(int ver, int pos, int val) {
		assert(rangeL <= rangeR);
		root.emplace_back(assign(root[ver], rangeL, rangeR, pos, pos, val));
	}

	inline void assign(int ver, int l, int r, int val) {
		assert(rangeL <= rangeR);
		root.emplace_back(assign(root[ver], rangeL, rangeR, l, r, val));
	}

	inline void add(int ver, int pos, int val) {
		assert(rangeL <= rangeR);
		root.emplace_back(add(root[ver], rangeL, rangeR, pos, pos, val));
	}

	inline void add(int ver, int l, int r, int val) {
		assert(rangeL <= rangeR);
		root.emplace_back(add(root[ver], rangeL, rangeR, l, r, val));
	}

	inline int query(int ver, int pos) {
		assert(rangeL <= rangeR);
		// root.emplace_back(root[ver]);
		return query(root[ver], rangeL, rangeR, pos, pos);
	}

	inline int query(int ver, int l, int r) {
		assert(rangeL <= rangeR);
		// root.emplace_back(root[ver]);
		return query(root[ver], rangeL, rangeR, l, r);
	}
};
```

## 怎么用

```cpp
int n;
cin >> n;
vector<int> a(n + 1);
for (int i = 1; i <= n; i++) cin >> a[i];

PresidentSegmentTree pst(n);
pst.build(a, 1, n);            // 版本 0

pst.add(0, 1, n, 5);           // 基于版本 0：整段 +5，生成版本 1
pst.assign(1, 2, 4, 0);        // 基于版本 1：把 [2,4] 赋值为 0，生成版本 2

cout << pst.query(2, 1, n) << "\n";  // 在版本 2 上查询区间和
cout << pst.query(0, 3) << "\n";     // 在版本 0 上单点查询，老版本不受影响
```

## 说明

区间版比单点版多了两件事：

1. `push_down` 时**先克隆孩子**再下传标记，否则会改到别的版本共享的结点；
2. 结点上同时挂着 `is_assign` / `lazy_assign` / `lazy_add`，`push_up` 和 `query` 都要先判断「这一段是不是被整体赋值过」，不能只看孩子的和。

区间被完全覆盖时直接给结点打标记、不再往下走，孩子继续和其他版本共享——这就是 $O(\log n)$ 的来处。

## 易错点

- `push_down` 里克隆孩子是主线，漏掉会让所有版本串味。
- `query` 没有完全覆盖时**不** `push_down`，而是用 `max(l, lb)`、`min(r, ub)` 算出重叠长度，靠 `is_assign` / `lazy_add` 现场补上本结点的贡献。
- `build` 之后 `root` 只有一个元素（版本 0），之后每次 `add` / `assign` 都 `emplace_back` 出新版本。
- 区间赋值会把 `lazy_add` 清 0；如果两段标记都要保留，先赋值再加法——顺序反了就错了。
<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
