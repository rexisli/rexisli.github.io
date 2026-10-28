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
math: true
---

## 摘要

给线段树加了懒标记之后，「路径复制」就不够了。原因很直接：@@push_down@@ 要**写子结点**，而子结点往往正被旧版本共享着——一次下传就把历史版本悄悄改坏。本文给出两处修正，它们配合才能得到正确的可持久化懒标记线段树：

1. **写时复制**：@@push_down@@ 在动左右儿子之前先 @@clone_node@@；
2. **标记计入自身值**（append-tag）：结点 @@dat@@ 里预先算进自己的标记，于是部分覆盖的查询可以沿途累加父标记、根本不需要下传。

前者保证不污染历史，后者把「必须下传」的需求削掉一大半。文末给出代价估计与几个只在多版本交叉查询时才暴露的坑。

## 1 问题与记号

维护序列的 $\mathrm{Add}(l,r,v)$、$\mathrm{Assign}(l,r,v)$、$\mathrm{Query}(l,r)=\sum$，并且每次修改都从某个**历史版本**出发，产生一个新版本，老版本永远可查。

记号同前两篇：$T(u)$ 为结点值域，$L(u)=|T(u)|$，$\mathrm{dat}(u)$ 为结点值。标记三元组：$\mathrm{is\_assign}$、$\mathrm{lazy\_assign}$、$\mathrm{lazy\_add}$，语义为「先赋值 $\mathrm{lazy\_assign}$，再整体加 $\mathrm{lazy\_add}$」。

## 2 直接照搬为什么坏

设版本 $V_1$ 与 $V_2$ 共享结点 $u$（$u$ 带标记、其子结点也是共享的）。现在对 $V_2$ 做一次**部分覆盖**的区间加：

1. @@add@@ 在路径上 @@clone_node@@，得到副本 $u'$；
2. @@push_down(u')@@ —— 若它直接把 $\mathrm{lazy}$ 写进 @@tr[u'.lc]@@，而 @@u'.lc@@ 仍是**旧下标**（与 $V_1$ 共用），那么 $V_1$ 的这棵子树就被改了。

**引理 1（污染）.** 若 @@push_down@@ 不复制子结点就写入，则存在版本对 $(V_1,V_2)$ 与操作序列，使 $V_1$ 的查询结果改变。

*证明.* 上例即构造：$V_1$ 的结点 $u$ 的子结点值被加上 $v$，而 $V_1$ 的 $\mathrm{dat}(u)$ 未变，于是 $V_1$ 在 $u$ 以下的部分覆盖查询结果偏移。$\square$

这个 bug 的可怕之处：单版本测试、随机对拍（在同一版本上连续操作）**全部通过**，只有「从旧版本再查询」时才错。

## 3 修正一：下传前复制

@@push_down@@ 的每一步写子结点之前，先 @@clone_node@@ 并更新父结点里的子指针：

```cpp
inline void push_down(int rt, int l, int r) {
	int mid = (l + r) >> 1;
	if (tr[rt].is_assign) {
		tr[rt].lc = clone_node(tr[rt].lc);   // 先复制，再写
		tr[rt].rc = clone_node(tr[rt].rc);
		apply_assign(tr[rt].lc, l, mid, tr[rt].lazy_assign);
		apply_assign(tr[rt].rc, mid + 1, r, tr[rt].lazy_assign);
		tr[rt].is_assign = false;
	}
	if (tr[rt].lazy_add) {
		tr[rt].lc = clone_node(tr[rt].lc);   // 同上
		tr[rt].rc = clone_node(tr[rt].rc);
		apply_add(tr[rt].lc, l, mid, tr[rt].lazy_add);
		apply_add(tr[rt].rc, mid + 1, r, tr[rt].lazy_add);
		tr[rt].lazy_add = 0;
	}
}
```

这样写的是本版本私有的新结点，历史版本仍指向原来的结点。

## 4 修正二：dat 记入自身标记

若结点有标记 $\mathrm{lazy\_add}$ 而子结点还没收到，那么该结点的 $\mathrm{dat}$ 应当是「子树真实值 + 自身标记 × 长度」。@@push_up@@ 因此分两支：

```cpp
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
```

- 若 @@is_assign@@：整个结点值相同，$\mathrm{dat}=(\mathrm{lazy\_assign}+\mathrm{lazy\_add})\cdot L$，与子树无关；
- 否则：$\mathrm{dat}=\mathrm{dat}(lc)+\mathrm{dat}(rc)+\mathrm{lazy\_add}\cdot L$$。

**引理 2.** 上述 @@push_up@@ 在两种情形下都给出 $\mathrm{dat}(u)$ 的正确值。

*证明.* 情形一由赋值语义直接得出。情形二：子结点值已各自记入其自身标记（归纳假设），尚未计入的只有 $u$ 自己的 $\mathrm{lazy\_add}$，它对每个元素贡献 $\mathrm{lazy\_add}$，共 $L$ 份。$\square$

这个做法常被叫**标记永久化**的思路：标记不急着下传，而是永久留在结点上、在每个用到它的地方补算出贡献。

## 5 查询：沿途累加

既然标记不一定已下传，@@query@@ 必须自己补：

```cpp
int query(int rt, int l, int r, int lb, int ub) {
	if (rt == -1) return 0;
	if (lb <= l && r <= ub) return tr[rt].dat;
	int L = max(l, lb), R = min(r, ub), len = R - L + 1;
	if (tr[rt].is_assign) return (tr[rt].lazy_assign + tr[rt].lazy_add) * len;
	int res = 0, mid = (l + r) >> 1;
	if (lb <= mid) res += query(tr[rt].lc, l, mid, lb, ub);
	if (mid < ub)  res += query(tr[rt].rc, mid + 1, r, lb, ub);
	if (tr[rt].lazy_add) res += tr[rt].lazy_add * len;
	return res;
}
```

**引理 3.** 设结点 $u$ 值域被查询区间部分覆盖，重叠长度 $\mathrm{len}$，则

$$\mathrm{Query}\cap T(u)=\text{递归结果}+\mathrm{lazy\_add}(u)\cdot\mathrm{len},$$

若 $u$ 带 @@is_assign@@，则直接返回 $(\mathrm{lazy\_assign}+\mathrm{lazy\_add})\cdot\mathrm{len}$。

*证明.* 以 $u$ 为界把查询区间切成完全覆盖部分与部分覆盖部分；完全覆盖部分已被 $\mathrm{dat}(u)$ 计入（第 4 节引理 2），部分覆盖部分递归下去，唯一未计入的是 $\mathrm{lazy\_add}(u)$ 对重叠段的贡献，即 $\mathrm{lazy\_add}(u)\cdot\mathrm{len}$。$\square$

注意顺序：递归结果与 $\mathrm{lazy\_add}\cdot\mathrm{len}$ 相加放在返回前，与「先累加再递归」等价，但前者少一层参数。

## 6 不变量与代价

**不变量 I（双份记录）.** 对每个可达结点：$\mathrm{dat}(u)$ 已计入 $u$ 自身标记的贡献；而子树中尚未收到该标记的结点不知道它。查询时由第 5 节补齐。

**定理.** 第 3、4、5 节的三处修正合起来，使任意版本上的任意操作与暴力一致。

*证明提纲.* 对操作序列归纳：@@add@@/@@assign@@ 沿路径复制并维护 I（引理 2）；@@push_down@@ 复制后不破坏任何历史版本（引理 1 的反面）；查询由引理 3 补齐。$\square$

**代价.**

| 项 | 普通懒标记线段树 | 可持久化版 |
| --- | --- | --- |
| 每操作新结点 | 0 | $O(\log n)$ |
| @@push_down@@ 额外复制 | 0 | 至多 2 |
| 单次操作常数 | 1× | 约 2~3× |
| 空间 | $O(n)$ | $O((n+m)\log n)$ |

## 7 实现

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

## 8 使用

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

版本号就是 @@root@@ 的下标：@@build@@ 产生版本 0，每次 @@add@@/@@assign@@ 追加一个版本。**老版本随时可查，且不受新操作影响**——这正是可持久化的验收标准。

## 9 实现细节与易错点

- **@@push_down@@ 里漏 @@clone_node@@**：本文最核心的坑（引理 1）。单版本测试永远发现不了，必须写「老版本再查询」的对拍。
- **$\mathrm{apply\_add}$ 遇到 @@is_assign@@ 要把 $v$ 并入 @@lazy_assign@@**，而不是写进 @@lazy_add@@：否则「赋值后再加」被拆成两个标记，赋值标记一覆盖就丢。
- **@@push_up@@ 必须带 @@l,r@@**：要乘长度，这是与普通线段树签名不同之处。
- **@@build@@ 时用 @@root.back() = build(...)@@**：构造函数里已经压了一个占位根（@@emplace_back(-1)@@），直接 @@emplace_back@@ 会多出一个悬空版本。
- **@@query@@ 遇到 @@is_assign@@ 立即返回**，不要再递归——此时子树可能是 $-1$ 或旧值，递归会读到过期数据。
- **版本号 ≠ 操作次数**：只读操作不产生版本，别用循环变量当版本号。
- **@@vector@@ 扩容会让引用失效**：@@node &u = tr[i];@@ 之后又 @@emplace_back@@ 是未定义行为。本模板全部用下标访问，就是为了避开这一点。
- **内存**：约 $m\log n$ 个结点，每个含 5 个 @@int/bool@@ 字段 ≈ 20 B。$m=2\times10^5,\ n=2\times10^5$ 时要提前 @@reserve@@，否则扩容峰值会翻倍。
- **单点与区间重载并存**：@@assign(ver,pos,val)@@ 与 @@assign(ver,l,r,val)@@ 同名，参数个数不同；调用时注意别让 $int$ 被隐式转成区间。

## 10 扩展

**纯标记永久化（不下传）.** 把第 4、5 节推广到底：完全不下传，所有查询都沿途补算。省掉 @@push_down@@ 与两处 clone，常数更小，但要求可合并信息（如和、最值），对「先赋值后加」这种非交换标记也可行，只是补算必须能按标记的语义顺序合成。

**可持久化 01-Trie / 可持久化并查集.** 同一套「clone 路径 + 不改旧结点」。

**与势能线段树的不兼容.** 区间取 $\min$、区间取 $\gcd$ 这类需要访问子结点统计的算法（Segment Tree Beats）在共享结构上代价极高：一次 rebase 要改很多结点，路径复制退化为大面积复制。

**持久化 + 二分答案.** 把「版本」当时间轴，二分历史时刻；配合第 $k$ 小的模板就是「历史第 $k$ 大」类问题。

## 11 参考

- OI Wiki：[可持久化线段树](https://oi-wiki.org/ds/persistent-seg/)、[线段树的标记永久化](https://oi-wiki.org/ds/seg/)
- 经典讨论：Codeforces 博客 *Persistent segment tree with lazy propagation*（push_down 复制与 append-tag 两种做法）
- 模板题：[P3834](https://www.luogu.com.cn/problem/P3834)（静态第 k 小）、[P2572 [SCOI2010] 序列操作](https://www.luogu.com.cn/problem/P2572)（赋值/翻转/查询，可加可持久化做「历史查询」）
- 《算法导论》第 14 章：结点增广与不变量

<!-- 原文：洛谷文章 u0xycgy6「线段树」 -->
