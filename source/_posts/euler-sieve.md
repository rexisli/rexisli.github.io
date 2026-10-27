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
---

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

## 怎么用

```cpp
Euler e(1000000);          // 预处理 1..1e6

auto& primes = e.get_prime();
auto& isp = e.get_isprime();

cout << primes.size() << "\n";   // 质数个数
if (isp[97]) cout << "97 is prime\n";

// 分解质因数（返回不同的质因子，不含指数）
vector<int> f = e.get_factor(360);   // {2, 3, 5}
```

## 说明

欧拉筛的复杂度是 $O(n)$，比埃氏筛的 $O(n \log \log n)$ 更紧。它的关键在那一行 `if (i % prime[j] == 0) break;`：

保证每个合数**只被它最小的质因子筛掉一次**。设 $i = p \cdot k$（$p$ 是 $i$ 的最小质因子），那么 $i \cdot p_j$ 在 $p_j = p$ 之前不会被访问，之后也不该再访问——因为在 $p_j > p$ 时，$i \cdot p_j$ 的最小质因子是 $p$ 而不是 $p_j$，它应该留给那个更小的质因子去筛。

`get_factor` 用质数表试除，只试到 $pri^2 > num$ 就停：剩下的 `num > 1` 时它自己就是最后一个质因子。所以分解单个数的复杂度是 $O(\pi(\sqrt n))$，而不是 $O(\sqrt n)$。

## 易错点

- `isprime` 用 `vector<bool>`（位压缩）省内存，代价是不能取引用进 `auto&` 之外的循环里做非 const 操作——要改就直接 `e.get_isprime()` 不行，只能读。
- 构造函数里 `isprime.assign(MAXN + 10, true)` 多留了 10 的余量，防止调用方传 `n` 之后在把 `num` 分解时越界。
- 分解质因数返回的是 **不同质因子**，指数需要自己再数一遍。
- `i * prime[j]` 用 `1LL * i * prime[j]` 防溢出（虽然 $i \le n$ 时一般不会，但在 $n$ 接近 `INT_MAX` 的场合会）。

<!-- 原文：洛谷文章 qnwxwf2m「数学」 -->
