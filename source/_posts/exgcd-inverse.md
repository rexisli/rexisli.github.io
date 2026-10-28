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
math: true
---

## 摘要

欧几里得算法给出 $g=\gcd(a,b)$，**扩展**欧几里得同时给出贝祖系数 $(x,y)$ 使

$$ax+by=g.$$

有了它，模意义下的逆元只是「方程右边取 1」的特例：$ax\equiv 1\pmod p \iff ax+py=1$，即 $g=\gcd(a,p)=1$ 时存在解，$x$ 就是逆元。

本文的重点不在公式，而在**那个把 $x,y$ 交换的递归**——它不是技巧，而是把二级递归的尾处理压成一行。同时给出终止性、正确性归纳，以及负数、溢出、$\gcd\neq1$ 三个真实会踩的坑。

## 1 问题与记号

求 $(x,y)$ 使 ${ax+by=g,\quad g=\gcd(a,b).}$

**裴蜀定理**（不证明，见 [OI Wiki](https://oi-wiki.org/math/number-theory/bezouts/)）：上式有整数解当且仅当 $g\mid c$，此处 $c=g$ 自然满足。解不唯一：若 $(x_0,y_0)$ 是一组解，则全体解为

$$x=x_0+\frac{b}{g}t,\qquad y=y_0-\frac{a}{g}t.$$

## 2 递归推导

设 $b\neq 0$。由带余除法 $a=qb+r$（$q=\lfloor a/b\rfloor$，$r=a\bmod b$）。若已求得 $b x'+r y'=g$，把它改写成 $a,b$ 的组合：

$$g=bx'+ry'=bx'+(a-qb)y'=ay'+b(x'-qy').$$

对照 $ax+by=g$ 得到

$$x=y',\qquad y=x'-qy'.$$

**这就是全部推导。** 记住它是「用 $(a,b)$ 的下一层系数表示上一层」。

## 3 交换引用为什么能省一行

按第 2 节，正常的二级递归要写：

```cpp
int exgcd(int a, int b, int &x, int &y) {
	if (!b) { x = 1, y = 0; return a; }
	int x1, y1;
	int d = exgcd(b, a % b, x1, y1);   // 先拿到 (x', y')
	x = y1;                            // 再按第 2 节修正
	y = x1 - a / b * y1;
	return d;
}
```

模板里则写成 @@exgcd(b, a % b, y, x)@@ —— 传参时就把 $x$ 和 $y$ 的位置对调了。于是子问题把 $y'$ 写进调用方的 $x$、把 $x'$ 写进调用方的 $y$；返回后调用方只需修正 $y$：

```cpp
int exgcd(int a, int b, int &x, int &y) {
	if (!b) { x = 1, y = 0; return a; }
	int d = exgcd(b, a % b, y, x);     // 传参时就对调
	y -= a / b * x;                    // 只剩一个修正
	return d;
}
```

**引理 1.** 这两种写法等价。

*证明.* 第一种写法里调用方得到 $(X,Y)=(x',y')$ 后再做 $x=Y,\ y=X-qY$；第二种写法子调用直接令本层 $x=y'$、本层 $y=x'$，再做 $y\leftarrow y-qx=x'-qy'$。两者输出相同。$\square$

## 4 终止性与正确性

**引理 2（终止）.** 递归序列 $(a,b)\to(b,a\bmod b)$ 在有限步内到达 $b=0$。

*证明.* 若 $b>0$ 则 $a\bmod b<b$，第二分量严格下降，非负整数不能无限下降。$\square$

**引理 3（归纳）.** 若下层返回 $(x',y')$ 满足 $bx'+ry'=g$，则上层返回的 $(x,y)$ 满足 $ax+by=g$。

*证明.* 见第 2 节等式链；且 $\gcd(a,b)=\gcd(b,r)$ 保住了 $g$。$\square$

**定理.** @@exgcd(a,b,x,y)@@ 返回 $gcd(a,b)$，并把一组贝祖系数写入 $x,y$。

*证明.* 由引理 2 递归终止于 @@b=0@@，此时 $x=1,y=0$ 满足 $a\cdot1+0=a=g$（基例成立）；再沿返回链由引理 3 逐层成立。$\square$

## 5 复杂度

递归深度为欧几里得步数，由**拉梅定理**上界为 $O(\log \min(a,b))$（最坏情形是相邻斐波那契数）。每层常数次整数运算，故时间 $O(\log\min(a,b))$，空间为递归栈同阶（$<10^9$ 时不超过 45 层）。

## 6 实现

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

## 7 求逆元

要求 $a^{-1}\bmod p$：解 $ax+py=1$。

1. 调 @@exgcd(a, p, x, y)@@，得 $d=\gcd(a,p)$；
2. 若 $d\neq 1$，逆元不存在（$a$ 与 $p$ 不互素）；
3. 否则 $x$ 是一组解，归一化到 $[0,p)$：$x\leftarrow (x\bmod p+p)\bmod p$。

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

**为什么必须归一化.** 贝祖系数只是「一组」解（第 1 节通解），$x$ 完全可能是负数或超过 $p$；而取模运算在 C++ 中对负数给出负余数，所以要用两次取模（或先 @@x %= p; if (x < 0) x += p;@@）。

**也存在不用 exgcd 的求法.** 若 $p$ 是素数且 $a\not\equiv0$，由费马小定理 $a^{-1}\equiv a^{p-2}$，一次快速幂即可（常数更小，但要求素模）。批量求 $1..n$ 的逆元可用递推 $\mathrm{inv}[i]=p-\lfloor p/i\rfloor\cdot\mathrm{inv}[p\bmod i]\bmod p$，$O(n)$。

## 8 实现细节与易错点

- **负数输入会得到负的 $g$**：C++ 的 $\%$ 保留被除数符号（$-7\%3=-1$），模板默认 $a,b\ge0$。要支持负数就先取绝对值、最后按符号修正，或直接用 $\mathrm{std::gcd}$ 另行处理。
- **$y -= a / b * x;$ 可能溢出**：$a/b$ 与 $x$ 都在 @@int@@ 范围时乘积可达 $10^{18}$，需转 @@long long@@ 或改用取模版本。
- **$d\neq1$ 时不存在逆元**，别忘了判；直接返回 $x$ 会得到一个「看似有效」的错值。
- **归一化要用两次取模**（或加一次 $p$），见上一节。
- **$p=1$ 退化**：此时所有数同余于 0，逆元无意义，通常题目保证 $p>1$ 且为素数。
- **不要用 $x,y$ 的初值**：模板会完全覆盖它们，调用前无需初始化。
- **基例 @@b=0@@ 的返回值**：$\gcd(a,0)=|a|$，模板返回的是未取绝对值的 $a$，这正是上面「负数」坑的来源。

## 9 扩展

**解 $ax+by=c$.** 先求 $ax_0+by_0=g$；若 $g\nmid c$ 则无解，否则一组解为 $(x_0c/g,\ y_0c/g)$，通解同第 1 节。

**线性同余 $ax\equiv b\pmod p$.** 等价于 $ax+py=b$，条件 $\gcd(a,p)\mid b$；解出后模 $p/\gcd$ 归一。

**中国剩余定理.** 两两互素时直接合并；不互素（EXCRT）时每一步都是一个 $ax\equiv b \pmod p$ 的判定 + 解，反复调用 exgcd。

**和欧拉函数的关系.** $a^{\varphi(p)}\equiv1$（$\gcd(a,p)=1$），所以 $a^{\varphi(p)-1}$ 也是逆元；exgcd 是更通用的做法（不要求知道 $\varphi$）。

## 10 参考

- OI Wiki：[裴蜀定理](https://oi-wiki.org/math/number-theory/bezouts/)、[逆元](https://oi-wiki.org/math/number-theory/inverse/)、[扩展欧几里得算法](https://oi-wiki.org/math/number-theory/gcd/#扩展欧几里得算法)
- 模板题：[P1082 [NOIP2012] 同余方程](https://www.luogu.com.cn/problem/P1082)、[P5656 【模板】二元一次不定方程](https://www.luogu.com.cn/problem/P5656)、[P1495 【模板】中国剩余定理](https://www.luogu.com.cn/problem/P1495)
- 《算法导论》第 31 章（数论算法的正确性证明）

<!-- 原文：洛谷文章 qnwxwf2m「数学」 -->
