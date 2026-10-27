---
title: 主席树（单点修改，区间第 k 小）
date: 2026-10-27 23:40:00
categories:
  - 模板
  - 线段树
tags:
  - 主席树
  - 可持久化线段树
---

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

## 怎么用

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

## 说明

主席树是「不修改老结点，复制一份再改」的线段树。每次插入只沿一条根到叶的路径复制 $O(\log n)$ 个结点，所以 $n$ 次插入总共 $O(n \log n)$ 个顶点。

区间第 $k$ 小用的是**前缀和差分**：`root[r]` 减 `root[l-1]` 之后，一棵子树里剩下的计数就是值落在这段值域里的元素个数。看左子树的 `cnt`：$k \le cnt$ 就进左儿子，否则 $k \mathrel{-}= cnt$ 进右儿子。

这个类里的 `assign` / `add` 都是**单点**版本（内部把 `(pos, pos)` 传下去），要区间修改请用「主席树（区间加、区间赋值）」那篇。

## 易错点

- 版本号从 0 开始：`build` 之后 `root[0]` 才有值；之后每 `add` 一次，`root` 末尾多一个版本，下标正好是版本号。
- `query_kth` 返回的是**离散化后的下标**，一定要 `b[pos - 1]` 换回原值。
- `build(a, 1, n)` 会用它建满整棵树，所以传入的 `a` 必须是你想要的初值；如果只想要空树，传等长的全 0 数组。
- `clone_node(-1)` 会新建空结点，不用额外判空。
<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
