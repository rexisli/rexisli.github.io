---
title: 欧拉筛（线性筛 + 因数分解）
date: 2026-10-27 23:45:00
categories:
  - 模板
  - 数学
tags:
  - 数论
  - 筛法
  - 质数
math: true
---

## 摘要

埃氏筛的问题不是「筛不干净」，而是**筛得太多次**：每个合数被它的每个素因子各筛一遍，总代价 $\sum_{p\le n}\frac{n}{p}=O(n\log\log n)$。欧拉筛（线性筛）用一句话把它压到 $O(n)$：

> 让每个合数 **只被它的最小素因子** 筛掉一次。

实现上只多了一行 @@if (i % prime[j] == 0) break;@@，但这一行的位置、它与「标记」的先后、以及它为什么恰好保证唯一性，值得完整说清。本文给出不变式、唯一性与线性复杂度证明，并说明 @@get_factor@@ 的正确边界（只返回不同素因子、试除到 $p^2>num$）。

## 1 记号

- $\mathbb{P}$ 素数列，$p_j$ 为第 $j$ 个素数（从 $2$ 起）；$\mathrm{spf}(x)$ 为 $x$ 的最小素因子。
- 筛完 $[1,n]$ 后：@@isprime[v]@@ 表示 $v$ 是否为素数，@@prime@@ 升序存放所有 $\le n$ 的素数。

## 2 不变式

升序枚举 $i=2..n$，内层按 $j=0,1,2,\dots$ 枚举已收集的素数，标记 $i\cdot p_j$ 为合数，遇到 $p_j\mid i$ 立即 @@break@@。

**引理 1（内层不变量）.** 设已收集素数为 $p_0<p_1<\dots< p_{k-1}$，则内层实际枚举的 $p_j$ 满足

$$p_j\le \mathrm{spf}(i)\quad(\text{含等号时} \;p_j=\mathrm{spf}(i)\;\text{且随后 break}).$$

*证明.* 枚举是升序的，一旦 $p_j\mid i$ 就停；而 $p_j\mid i$ 且 $p_j$ 为素数，等价于 $p_j=\mathrm{spf}(i)$（更大的素因子整除 $i$ 时也仍会先碰到 $\mathrm{spf}(i)$）。故枚举到的都满足 $p_j\le\mathrm{spf}(i)$。$\square$

**引理 2（被筛合数的最小素因子）.** 对被标记的 $x=i\cdot p_j$，有 $\mathrm{spf}(x)=p_j$。

*证明.* 由引理 1，$p_j\le \mathrm{spf}(i)$，而 $\mathrm{spf}(x)=\min(\mathrm{spf}(i),p_j)=p_j$。$\square$

## 3 唯一性与线性

**定理 1（每个合数恰好被筛一次）.** 任意合数 $x$（$2\le x\le n$）在算法中恰被标记一次。

*证明.* 设 $p=\mathrm{spf}(x)$，写法唯一：$x=p\cdot m$，其中 $m=x/p$ 且 $\mathrm{spf}(m)\ge p$（否则 $m$ 有更小素因子，与 $p$ 最小矛盾）。

- *至少一次*：当 $i=m$ 时，$p$ 已在 $prime@$ 中（因为 $p\le m\le x\le n$），且 $p\le \mathrm{spf}(m)$ 由引理 1 该组合会被枚举到，于是 $x$ 被标记。
- *至多一次*：若 $x=i\cdot p_j$ 被标记，由引理 2 得 $p_j=\mathrm{spf}(x)=p$，于是 $i=x/p=m$ 唯一确定。

$\square$

**定理 2（时间 $O(n)$）.** 总标记次数 $\le n$。

*证明.* 每个被标记的 $x$ 互不相同（定理 1），且 $x\le n$，故总次数 $\le n-1$。外层 $n$ 次循环、内层均摊常数。$\square$

对比埃氏筛：$O(n\log\log n)$，差距在 $n$ 大时明显（$n=10^8$ 时约 3~4 倍常数差）。

## 4 复杂度与空间

时间 $O(n)$；空间 $O(n)$，其中 @@vector<bool>@@ 是位压缩（$10^8$ 约 12.5 MB），@@vector<int>@@ 素数表约 $\pi(n)\approx n/\ln n$ 个数。

@@MAXN + 10@@ 的冗余是为了容纳边界，与复杂度无关。

## 5 实现

```cpp
class Euler {
private:
	int MAXN = 0;
	vector<int> prime;
	vector<bool> isprime;
	inline void build() {
		isprime.assign(MAXN + 10, true);
		isprime[0] = isprime[1] = false;
		for (int i = 2; i <= MAXN; i++) {
			if (isprime[i])	prime.push_back(i);
			for (int j = 0; j < prime.size() && i * prime[j] <= MAXN; j++) {
				isprime[1LL * i*prime[j]] = false;
				if (i % prime[j] == 0) break;
			}
		}
	}
public:
	Euler(int maxn): MAXN(maxn) {
		build();
	}

	inline vector<int> get_factor(int num) {
		vector<int> res;
		for (auto pri : prime) {
			if (num == 1 || 1LL * pri * pri > num) break;
			if (num % pri == 0) res.push_back(pri);
			while (num % pri == 0) num /= pri;
		}
		if (num > 1)res.emplace_back(num);
		return res;
	}

	const vector<int>& get_prime() const {
		return prime;
	}
	const vector<bool>& get_isprime() const {
		return isprime;
	}
};
```

注意 @@isprime[1LL * i * prime[j]]@@ 里的 $1LL$：$i$ 与 $p_j$ 都是 @@int@@，直接相乘在 $n$ 接近 $2^{31}$ 时会溢出（虽然此时内存也放不下，但边界测试里常见）。

## 6 使用

```cpp
Euler e(1000000);          // 预处理 1..1e6

auto& primes = e.get_prime();
auto& isp = e.get_isprime();

cout << primes.size() << "\n";   // 质数个数
if (isp[97]) cout << "97 is prime\n";

// 分解质因数（返回不同的质因子，不含指数）
vector<int> f = e.get_factor(360);   // {2, 3, 5}
```

**@@get_factor@@ 的边界**值得单独说：循环条件是 $num=1$ 或 $p_j^2>num$ 时退出；此时若 $num>1$，剩下的 $num$ 本身必为素数，补进答案。返回的是**不同素因子**（不带指数），因为内层用 @@while (num % pri == 0) num /= pri;@@ 把指数除净了。

## 7 实现细节与易错点

- **1 既不是素数也不是合数**：必须显式 @@isprime[0] = isprime[1] = false@@。
- **@@break@@ 的位置**：必须在「标记」之后。写在标记之前会漏掉 $i\cdot p_j$ 这个数（它恰是 $p_j=\mathrm{spf}(i)$ 的那一项）。
- **@@break@@ 的条件是 $i \% p_j == 0$**（$p_j\mid i$），不是 $x \% p_j==0$——后者恒真，退化成埃氏筛的常数倍。
- **$i\cdot p_j$ 溢出**：用 $1LL$ 提升，或先判 $p_j > n/i$ 再乘。
- **@@get_factor(1)@@ 返回空**，调用方要能接受；@@get_factor@@ 依赖筛的范围，$num\le MAXN$ 时正确，超过时需要先用别的办法（Pollard-Rho）分解。
- **@@get_factor@@ 只给素因子不给重数**：要重数就在 @@while@@ 里计数。
- **@@vector<bool>@@ 不是真正的容器**：@@auto \&b = isprime[v];@@ 拿不到引用（代理对象），取地址也不合法。
- **重复调用 @@build@@** 会累积 $prime$，需要先 @@clear@@（本实现只在构造里调用一次）。

## 8 扩展

**线性筛顺带求积性函数.** 用同一套枚举，令：

- $\varphi(x)$：$\varphi(p)=p-1$；$p\mid i$ 时 $\varphi(ip)=p\varphi(i)$，否则 $(p-1)\varphi(i)$；
- $\mu(x)$：$p\mid i$ 时 $\mu(ip)=0$，否则 $-\mu(i)$；
- 约数个数 $d(x)$ 需额外维护最小素因子的指数。

**分段筛.** $n$ 很大而询问区间 $[L,R]$ 很窄时，用 $\sqrt{R}$ 内的素数筛区间，空间 $O(R-L)$。

**大数分解.** $n>10^{12}$ 时筛法失效，改用 Miller–Rabin 判素 + Pollard–Rho 分解。

## 9 参考

- OI Wiki：[素数筛法](https://oi-wiki.org/math/number-theory/sieve/)、[最小素因子与线性筛](https://oi-wiki.org/math/number-theory/sieve/#线性筛)
- 模板题：[P3383 【模板】线性筛素数](https://www.luogu.com.cn/problem/P3383)、[P1835 素数密度](https://www.luogu.com.cn/problem/P1835)（分段筛）
- 《具体数学》第 4 章：整除性与素数分布的初等结论

<!-- 原文：洛谷文章 qnwxwf2m「数学」 -->
