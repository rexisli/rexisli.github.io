---
title: SegmentTree
date: 2026-04-26 15:42:30
tags: C++/模板
---
### 动态开点
```cpp 
template<typename T>
class SegmentTree {
	struct __attribute__((packed)) node {
		int lc = 0, rc = 0;
		T info = T(0);
		T lazy_assign = T(0);
		T lazy_add = T(0);
		bool is_assign = false;
	};

	vector<node> tree;
	int root = 0;
	int rangeL, rangeR;

	inline int create_node() {
		tree.emplace_back();
		return (int) tree.size() - 1;
	}

	inline void apply_assign(int rt, int l, int r, T val) {
		int len = r - l + 1;
		tree[rt].is_assign = true;
		tree[rt].lazy_assign = val;
		tree[rt].lazy_add = T(0);
		tree[rt].info = val * len;
	}

	inline void apply_add(int rt, int l, int r, T val) {
		int len = r - l + 1;
		tree[rt].lazy_add += val;
		tree[rt].info += val * len;
	}

	inline void push_up(int rt) {
		tree[rt].info = tree[tree[rt].lc].info + tree[tree[rt].rc].info;
	}

	inline void push_down(int rt, int l, int r) {
		int mid = (l + r) >> 1;
		if (!tree[rt].lc) tree[rt].lc = create_node();
		if (!tree[rt].rc) tree[rt].rc = create_node();

		if (tree[rt].is_assign) {
			apply_assign(tree[rt].lc, l, mid, tree[rt].lazy_assign);
			apply_assign(tree[rt].rc, mid + 1, r, tree[rt].lazy_assign);
			tree[rt].is_assign = false;
		}
		if (tree[rt].lazy_add != T(0)) {
			apply_add(tree[rt].lc, l, mid, tree[rt].lazy_add);
			apply_add(tree[rt].rc, mid + 1, r, tree[rt].lazy_add);
			tree[rt].lazy_add = T(0);
		}
	}

	int build(int rt, int l, int r, T *arr) {
		if (!rt) rt = create_node();
		if (l == r) {
			tree[rt].info = arr[l];
			return rt;
		}
		int mid = (l + r) >> 1;
		tree[rt].lc = build(tree[rt].lc, l, mid, arr);
		tree[rt].rc = build(tree[rt].rc, mid + 1, r, arr);
		push_up(rt);
		return rt;
	}

	int assign(int rt, int l, int r, int ql, int qr, T val) {
		if (!rt) rt = create_node();
		if (ql <= l && r <= qr) {
			apply_assign(rt, l, r, val);
			return rt;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (ql <= mid) tree[rt].lc = assign(tree[rt].lc, l, mid, ql, qr, val);
		if (mid + 1 <= qr) tree[rt].rc = assign(tree[rt].rc, mid + 1, r, ql, qr, val);
		push_up(rt);
		return rt;
	}

	int add(int rt, int l, int r, int ql, int qr, T val) {
		if (!rt) rt = create_node();
		if (ql <= l && r <= qr) {
			apply_add(rt, l, r, val);
			return rt;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (ql <= mid) tree[rt].lc = add(tree[rt].lc, l, mid, ql, qr, val);
		if (mid + 1 <= qr) tree[rt].rc = add(tree[rt].rc, mid + 1, r, ql, qr, val);
		push_up(rt);
		return rt;
	}

	T query(int rt, int l, int r, int ql, int qr) {
		if (!rt) return T(0);
		if (ql <= l && r <= qr) return tree[rt].info;
		push_down(rt, l, r);
		T ans = T(0);
		int mid = (l + r) >> 1;
		if (ql <= mid) ans += query(tree[rt].lc, l, mid, ql, qr);
		if (mid + 1 <= qr) ans += query(tree[rt].rc, mid + 1, r, ql, qr);
		return ans;
	}

public:
	SegmentTree() : rangeL(1), rangeR(0) {
		tree.emplace_back();
	}

	explicit SegmentTree(int l, int r) : rangeL(l), rangeR(r) {
		tree.emplace_back();
	}

	explicit SegmentTree(int n) : SegmentTree(1, n) {
	}

	void resize(int l, int r) {
		rangeL = l;
		rangeR = r;
		tree.clear();
		tree.emplace_back();
		root = 0;
	}

	void reserve(size_t n) {
		tree.reserve(n);
	}

	void build(T *arr, int l, int r) {
		rangeL = l;
		rangeR = r;
		tree.reserve(4 * (r - l + 1) + 5);
		root = build(0, rangeL, rangeR, arr);
	}

	void assign(int pos, T val) {
		root = assign(root, rangeL, rangeR, pos, pos, val);
	}

	void assign(int l, int r, T val) {
		root = assign(root, rangeL, rangeR, l, r, val);
	}

	void add(int pos, T val) {
		root = add(root, rangeL, rangeR, pos, pos, val);
	}

	void add(int l, int r, T val) {
		root = add(root, rangeL, rangeR, l, r, val);
	}

	T query(int l, int r) {
		return query(root, rangeL, rangeR, l, r);
	}
};

SegmentTree<int> segtree;
```

### 普通线段树模板 赋值+区间加法+区间查询
```cpp 
const int mxn=1e5+10;
template <typename T>
class SegmentTree{
#define lc rt<<1
#define rc rt<<1|1
protected:
	T tree[mxn<<2]={};
	T lazy[mxn<<2][2]={};//0->赋值 1->加
	bool set_tag[mxn<<2]={};
	inline void push_up(int rt){
		tree[rt]=tree[lc]+tree[rc];
	}
	inline void push_down(int rt,int l,int r){
		if(l==r){
			return;
		}
		int mid=(l+r)>>1;
		if(set_tag[rt]){
			set_tag[lc]=true;
			set_tag[rc]=true;
			lazy[lc][0]=lazy[rt][0];
			lazy[rc][0]=lazy[rt][0];
			lazy[lc][1]=0;
			lazy[rc][1]=0;
			tree[lc]=lazy[rt][0]*(mid-l+1);
			tree[rc]=lazy[rt][0]*(r-mid);
			set_tag[rt]=false;
		}
		if(lazy[rt][1]){
			lazy[lc][1]+=lazy[rt][1];
			lazy[rc][1]+=lazy[rt][1];
			tree[lc]+=lazy[rt][1]*(mid-l+1);
			tree[rc]+=lazy[rt][1]*(r-mid);
			lazy[rt][1]=0;
		}
	}
public:
	void build(int rt,int l,int r,T* arr){
		if(l==r){
			tree[rt]=arr[l];
			return;
		}
		int mid=(l+r)>>1;
		build(lc,l,mid,arr);
		build(rc,mid+1,r,arr);
		push_up(rt);
	}
	void assign(int rt,int l,int r,int o_l,int o_r,T val){
		if(o_l<=l&&r<=o_r){
			set_tag[rt]=true;
			lazy[rt][0]=val;
			lazy[rt][1]=0;
			tree[rt]=val*(r-l+1);
			return;
		}
		push_down(rt,l,r);
		int mid=(l+r)>>1;
		if(o_l<=mid){
			assign(lc,l,mid,o_l,o_r,val);
		}
		if(mid+1<=o_r){
			assign(rc,mid+1,r,o_l,o_r,val);
		}
		push_up(rt);
	}
	void add(int rt,int l,int r,int o_l,int o_r,T val){
		if(o_l<=l&&r<=o_r){
			lazy[rt][1]+=val;
			tree[rt]+=val*(r-l+1);
			return;
		}
		push_down(rt,l,r);
		int mid=(l+r)>>1;
		if(o_l<=mid){
			add(lc,l,mid,o_l,o_r,val);
		}
		if(mid+1<=o_r){
			add(rc,mid+1,r,o_l,o_r,val);
		}
		push_up(rt);
	}
	T query(int rt,int l,int r,int o_l,int o_r){
		if(o_l<=l&&r<=o_r){
			return tree[rt];
		}
		int mid=(l+r)>>1;
		push_down(rt,l,r);
		T res=0;
		if(o_l<=mid){
			res+=query(lc,l,mid,o_l,o_r);
		}
		if(mid+1<=o_r){
			res+=query(rc,mid+1,r,o_l,o_r);
		}
		return res;
	}
};
SegmentTree<int> segtree;
```
### 乘法
```cpp 
int mod=1;
template <typename T>
class SegmentTree {
	struct node {
		int lc = -1, rc = -1;
		T info = 0;
		T lazy_assign = 0;
		T lazy_mul = 1;
		T lazy_add = 0;
		bool is_assign = false;
	};
	
	vector<node> tree;
	int root = 0;
	int rangeL, rangeR;
	
	inline int create_node() {
		tree.emplace_back();
		return tree.size() - 1;
	}
	
	inline void check_child(int rt) {
		if (tree[rt].lc == -1) {
			int lc = create_node();
			tree[rt].lc = lc;
		}
		if (tree[rt].rc == -1) {
			int rc = create_node();
			tree[rt].rc = rc;
		}
	}
	
	inline void apply_assign(int rt, int l, int r, T val) {
		val %= mod;
		int len = r - l + 1;
		tree[rt].is_assign = true;
		tree[rt].lazy_assign = val;
		tree[rt].lazy_mul = 1;
		tree[rt].lazy_add = 0;
		tree[rt].info = val * len;
		tree[rt].info %= mod;
	}
	
	inline void apply_mul(int rt, int l, int r, T val) {
		val %= mod;
		if (tree[rt].is_assign) {
			tree[rt].lazy_assign *= val;
			tree[rt].lazy_assign %= mod;
			tree[rt].info *= val; // 等价于 (lazy_assign * len) * val
			tree[rt].info %= mod;
		} else {
			tree[rt].lazy_mul *= val;
			tree[rt].lazy_mul %= mod;
			tree[rt].lazy_add *= val;
			tree[rt].lazy_add %= mod;
			tree[rt].info *= val;
			tree[rt].info %= mod;
		}
	}
	
	inline void apply_add(int rt, int l, int r, T val) {
		val %= mod;
		int len = r - l + 1;
		if (tree[rt].is_assign) {
			tree[rt].lazy_assign += val;
			tree[rt].lazy_assign %= mod;
		} else {
			tree[rt].lazy_add += val;
			tree[rt].lazy_add %= mod;
		}
		tree[rt].info += val * len;
		tree[rt].info %= mod;
	}
	
	inline void push_up(int rt) {
		tree[rt].info = tree[tree[rt].lc].info + tree[tree[rt].rc].info;
		tree[rt].info %= mod;
	}
	
	inline void push_down(int rt, int l, int r) {
		check_child(rt);
		int mid = (l + r) >> 1;
		// 优先级：赋值 > 乘法 > 加法
		if (tree[rt].is_assign) {
			apply_assign(tree[rt].lc, l, mid, tree[rt].lazy_assign);
			apply_assign(tree[rt].rc, mid + 1, r, tree[rt].lazy_assign);
			tree[rt].is_assign = false;
		}
		if (tree[rt].lazy_mul != 1) {
			apply_mul(tree[rt].lc, l, mid, tree[rt].lazy_mul);
			apply_mul(tree[rt].rc, mid + 1, r, tree[rt].lazy_mul);
			tree[rt].lazy_mul = 1;
		}
		if (tree[rt].lazy_add != 0) {
			apply_add(tree[rt].lc, l, mid, tree[rt].lazy_add);
			apply_add(tree[rt].rc, mid + 1, r, tree[rt].lazy_add);
			tree[rt].lazy_add = 0;
		}
	}
	
	void build(int rt, int l, int r, T* arr) {
		if (l == r) {
			tree[rt].info = arr[l] % mod;
			return;
		}
		check_child(rt);
		int mid = (l + r) >> 1;
		build(tree[rt].lc, l, mid, arr);
		build(tree[rt].rc, mid + 1, r, arr);
		push_up(rt);
	}
	
	void assign(int rt, int l, int r, int ql, int qr, T val) {
		if (ql <= l && r <= qr) {
			apply_assign(rt, l, r, val);
			return;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (ql <= mid) assign(tree[rt].lc, l, mid, ql, qr, val);
		if (mid + 1 <= qr) assign(tree[rt].rc, mid + 1, r, ql, qr, val);
		push_up(rt);
	}
	
	void mul(int rt, int l, int r, int ql, int qr, T val) {
		if (ql <= l && r <= qr) {
			apply_mul(rt, l, r, val);
			return;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (ql <= mid) mul(tree[rt].lc, l, mid, ql, qr, val);
		if (mid + 1 <= qr) mul(tree[rt].rc, mid + 1, r, ql, qr, val);
		push_up(rt);
	}
	
	void add(int rt, int l, int r, int ql, int qr, T val) {
		if (ql <= l && r <= qr) {
			apply_add(rt, l, r, val);
			return;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (ql <= mid) add(tree[rt].lc, l, mid, ql, qr, val);
		if (mid + 1 <= qr) add(tree[rt].rc, mid + 1, r, ql, qr, val);
		push_up(rt);
	}
	
	T query(int rt, int l, int r, int ql, int qr) {
		if (ql <= l && r <= qr) return tree[rt].info;
		push_down(rt, l, r);
		T ans = 0;
		int mid = (l + r) >> 1;
		if (ql <= mid){ 
			ans += query(tree[rt].lc, l, mid, ql, qr);
			ans %= mod;
		}
		if (mid + 1 <= qr){ 
			ans += query(tree[rt].rc, mid + 1, r, ql, qr);
			ans %= mod;
		}
		return ans;
	}
	
public:
	SegmentTree() : rangeL(1), rangeR(0) {
		tree.emplace_back();
	}
	
	explicit SegmentTree(int l, int r) : rangeL(l), rangeR(r) {
		tree.emplace_back();
	}
	
	explicit SegmentTree(int n) : SegmentTree(1, n) {}
	
	void resize(int l, int r) {
		rangeL = l;
		rangeR = r;
		tree.clear();
		tree.emplace_back();
	}
	
	void build(T* arr, int l, int r) {
		rangeL = l;
		rangeR = r;
		build(root, rangeL, rangeR, arr);
	}
	
	void assign(int pos, T val) {
		assign(root, rangeL, rangeR, pos, pos, val);
	}
	void assign(int l, int r, T val) {
		assign(root, rangeL, rangeR, l, r, val);
	}
	
	void mul(int pos, T val) {
		mul(root, rangeL, rangeR, pos, pos, val);
	}
	void mul(int l, int r, T val) {
		mul(root, rangeL, rangeR, l, r, val);
	}
	
	void add(int pos, T val) {
		add(root, rangeL, rangeR, pos, pos, val);
	}
	void add(int l, int r, T val) {
		add(root, rangeL, rangeR, l, r, val);
	}
	
	T query(int l, int r) {
		return query(root, rangeL, rangeR, l, r);
	}
};
SegmentTree<int> segtree;
```
### 额外添加平方操作
```cpp 
template <typename T>
class SegmentTree {
	struct Info {
		T sum = 0;
		T sq  = 0;   // 平方和
		
		Info() = default;
		Info(T s, T sq_) : sum(s), sq(sq_) {}
		
		Info operator+(const Info& rhs) const {
			return Info(sum + rhs.sum, sq + rhs.sq);
		}
	};
	
	struct node {
		int lc = -1, rc = -1;
		Info info;                // 当前区间和、平方和
		T lazy_assign = 0;
		T lazy_mul = 1;
		T lazy_add = 0;
		bool is_assign = false;
	};
	
	vector<node> tree;
	int root = 0;
	int rangeL, rangeR;
	
	inline int create_node() {
		tree.emplace_back();
		return tree.size() - 1;
	}
	
	inline void check_child(int rt) {
		if (tree[rt].lc == -1) {
			int lc = create_node();
			tree[rt].lc = lc;
		}
		if (tree[rt].rc == -1) {
			int rc = create_node();
			tree[rt].rc = rc;
		}
	}
	
	// 区间赋值：直接设置值，平方和 = val*val*len
	inline void apply_assign(int rt, int l, int r, T val) {
		int len = r - l + 1;
		tree[rt].is_assign = true;
		tree[rt].lazy_assign = val;
		tree[rt].lazy_mul = 1;
		tree[rt].lazy_add = 0;
		tree[rt].info.sum = val * len;
		tree[rt].info.sq  = val * val * len;
	}
	
	// 区间乘法
	inline void apply_mul(int rt, int l, int r, T val) {
		int len = r - l + 1;
		if (tree[rt].is_assign) {
			// 已有赋值标记，直接乘在赋值值上，重新计算 sum 和 sq
			tree[rt].lazy_assign *= val;
			T new_val = tree[rt].lazy_assign;
			tree[rt].info.sum = new_val * len;
			tree[rt].info.sq  = new_val * new_val * len;
		} else {
			tree[rt].lazy_mul *= val;
			tree[rt].lazy_add *= val;
			tree[rt].info.sum *= val;
			tree[rt].info.sq  *= val * val;
		}
	}
	
	// 区间加法
	inline void apply_add(int rt, int l, int r, T val) {
		int len = r - l + 1;
		if (tree[rt].is_assign) {
			// 已有赋值标记，直接加在赋值值上，重新计算 sum 和 sq
			tree[rt].lazy_assign += val;
			T new_val = tree[rt].lazy_assign;
			tree[rt].info.sum = new_val * len;
			tree[rt].info.sq  = new_val * new_val * len;
		} else {
			tree[rt].lazy_add += val;
			// sq' = sq + 2*val*sum + val*val*len
			tree[rt].info.sq += 2 * val * tree[rt].info.sum + val * val * len;
			tree[rt].info.sum += val * len;
		}
	}
	
	inline void push_up(int rt) {
		tree[rt].info = tree[tree[rt].lc].info + tree[tree[rt].rc].info;
	}
	
	inline void push_down(int rt, int l, int r) {
		check_child(rt);
		int mid = (l + r) >> 1;
		// 优先级：赋值 > 乘法 > 加法
		if (tree[rt].is_assign) {
			apply_assign(tree[rt].lc, l, mid, tree[rt].lazy_assign);
			apply_assign(tree[rt].rc, mid + 1, r, tree[rt].lazy_assign);
			tree[rt].is_assign = false;
		}
		if (tree[rt].lazy_mul != 1) {
			apply_mul(tree[rt].lc, l, mid, tree[rt].lazy_mul);
			apply_mul(tree[rt].rc, mid + 1, r, tree[rt].lazy_mul);
			tree[rt].lazy_mul = 1;
		}
		if (tree[rt].lazy_add != 0) {
			apply_add(tree[rt].lc, l, mid, tree[rt].lazy_add);
			apply_add(tree[rt].rc, mid + 1, r, tree[rt].lazy_add);
			tree[rt].lazy_add = 0;
		}
	}
	
	void build(int rt, int l, int r, T* arr) {
		if (l == r) {
			T val = arr[l];
			tree[rt].info = Info(val, val * val);
			return;
		}
		check_child(rt);
		int mid = (l + r) >> 1;
		build(tree[rt].lc, l, mid, arr);
		build(tree[rt].rc, mid + 1, r, arr);
		push_up(rt);
	}
	
	void assign(int rt, int l, int r, int ql, int qr, T val) {
		if (ql <= l && r <= qr) {
			apply_assign(rt, l, r, val);
			return;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (ql <= mid) assign(tree[rt].lc, l, mid, ql, qr, val);
		if (mid + 1 <= qr) assign(tree[rt].rc, mid + 1, r, ql, qr, val);
		push_up(rt);
	}
	
	void mul(int rt, int l, int r, int ql, int qr, T val) {
		if (ql <= l && r <= qr) {
			apply_mul(rt, l, r, val);
			return;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (ql <= mid) mul(tree[rt].lc, l, mid, ql, qr, val);
		if (mid + 1 <= qr) mul(tree[rt].rc, mid + 1, r, ql, qr, val);
		push_up(rt);
	}
	
	void add(int rt, int l, int r, int ql, int qr, T val) {
		if (ql <= l && r <= qr) {
			apply_add(rt, l, r, val);
			return;
		}
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (ql <= mid) add(tree[rt].lc, l, mid, ql, qr, val);
		if (mid + 1 <= qr) add(tree[rt].rc, mid + 1, r, ql, qr, val);
		push_up(rt);
	}
	
	Info query(int rt, int l, int r, int ql, int qr) {
		if (ql <= l && r <= qr) return tree[rt].info;
		push_down(rt, l, r);
		int mid = (l + r) >> 1;
		if (qr <= mid) return query(tree[rt].lc, l, mid, ql, qr);
		if (ql > mid)  return query(tree[rt].rc, mid + 1, r, ql, qr);
		return query(tree[rt].lc, l, mid, ql, qr) + query(tree[rt].rc, mid + 1, r, ql, qr);
	}
	
public:
	SegmentTree() : rangeL(1), rangeR(0) {
		tree.emplace_back();
	}
	
	explicit SegmentTree(int l, int r) : rangeL(l), rangeR(r) {
		tree.emplace_back();
	}
	
	explicit SegmentTree(int n) : SegmentTree(1, n) {}
	
	void resize(int l, int r) {
		rangeL = l;
		rangeR = r;
		tree.clear();
		tree.emplace_back();
	}
	
	void build(T* arr, int l, int r) {
		rangeL = l;
		rangeR = r;
		build(root, rangeL, rangeR, arr);
	}
	
	void assign(int pos, T val) {
		assign(root, rangeL, rangeR, pos, pos, val);
	}
	void assign(int l, int r, T val) {
		assign(root, rangeL, rangeR, l, r, val);
	}
	
	void mul(int pos, T val) {
		mul(root, rangeL, rangeR, pos, pos, val);
	}
	void mul(int l, int r, T val) {
		mul(root, rangeL, rangeR, l, r, val);
	}
	
	void add(int pos, T val) {
		add(root, rangeL, rangeR, pos, pos, val);
	}
	void add(int l, int r, T val) {
		add(root, rangeL, rangeR, l, r, val);
	}
	
	T query_sum(int l, int r) {
		return query(root, rangeL, rangeR, l, r).sum;
	}
	
	T query_sq(int l, int r) {
		return query(root, rangeL, rangeR, l, r).sq;
	}
};
SegmentTree<int> segtree;
```