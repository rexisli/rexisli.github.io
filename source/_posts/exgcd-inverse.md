---
title: 扩展欧几里得（求逆元）
date: 2026-10-27 23:46:00
categories:
  - 模板
  - 数学
tags:
  - 数论
  - exgcd
  - 逆元
---

```cpp
int exgcd(int a, int b, int &x, int &y) {
	if (!b) {
		x = 1, y = 0;
		return a;
	}
	int d = exgcd(b, a%b, y, x);
	y -= a / b * x;
	return d;
}
```

## 怎么用

```cpp
int x, y;
int d = exgcd(a, b, x, y);    // d = gcd(a, b)，且 ax + by = d

// 求 a 在模 p 下的逆元（要求 gcd(a, p) == 1）
int inv, k;
if (exgcd((a % p + p) % p, p, inv, k) == 1)
    inv = (inv % p + p) % p;
else
    puts("无逆元");
```

## 说明

先说明它解的是什么：$ax + by = \gcd(a, b)$ 的一组整数解 $(x, y)$。设递归解出 $bx' + (a \bmod b)y' = \gcd$，而 $a \bmod b = a - \lfloor a/b \rfloor b$，代回去整理，就有 $x = y'$、$y = x' - \lfloor a/b \rfloor y'$。代码里 `exgcd(b, a%b, y, x)` 直接用引用交换了 `x`、`y` 的位置，所以后面只需要补一句 `y -= a / b * x`。

逆元的推导链：若 $\gcd(x, p) = 1$，由**裴蜀定理**存在整数 $y, k$ 使 $xy + pk = 1$；两边对 $p$ 取模，$pk \equiv 0$，于是 $xy \equiv 1 \pmod p$，这个 $y$ 就是 $x^{-1}$。所以「能不能求逆元」和「`exgcd` 返回的 gcd 是不是 1」是同一件事。

复杂度是 $O(\log \min(a, b))$（递归层数不超过斐波那契增长）。

## 易错点

- 这个模板默认 $a, b \ge 0$。传负数进去时 C++ 的 `%` 会保留被除数的符号，返回的 $\gcd$ 可能带负号，`d != 1` 的判断就会误判——负数先取绝对值再调。
- **逆元存在的充要条件是 $\gcd(x, p) = 1$**，即 $p$ 必须是质数或 $x$ 与 $p$ 互质；`exgcd` 返回的 `d != 1` 时直接无解。
- 负数取模要先规范化：`(a % p + p) % p`，否则 `a` 为负时 C++ 的 `%` 结果是负的。
- 求出的 `inv` 可能落在 $(-p, 0]$，最后再取一次模。
- 当 $p$ 是质数时，用费马小定理 + 快速幂求逆元 $a^{p-2}$ 更快写（$O(\log p)$），但 `exgcd` 对**合数模数**同样适用，这是它的优势。

<!-- 原文：洛谷文章 qnwxwf2m「数学」 -->
