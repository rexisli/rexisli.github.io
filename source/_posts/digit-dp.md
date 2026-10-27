---
title: 数位 DP（DigitDP 模板）
date: 2026-10-27 23:48:00
categories:
  - 模板
  - DP
tags:
  - DP
  - 数位DP
math: true
---

## 适用场景

统计 $[0, N]$ 里满足某个「逐位条件」的数的个数，比如「不含数字 4」「相邻两位不同」「数位和是 3 的倍数」。手动分情况讨论极其痛苦，数位 DP 的答案就是把每一类数「按位枚举一遍」，用记忆化避免重复。

模板长这样：从高位到低位枚举，用两个布尔量描述「当前还受不受上界约束」和「前面是不是全是前导零」。

## 参数含义

- `pos`：当前填到第几位（从 0 开始，高位在前）。
- `limit`：之前填的前缀是否**与上界完全一致**。若一致，当前位最多填到 `digit[pos]`；否则 0~9 随便填。
- `lead0`：当前是否**仍然全是前导零**。处理「数字本身从哪一位开始算」时要用到它。

## 为什么只在 `!limit` 时记忆化

`limit == true` 的状态只会沿着「贴着上界」的那一条链出现，数量是 $O(\text{digits})$，不记忆也不影响复杂度；而它的答案**依赖具体上界**，记忆下来会被别的数误用。所以只有 `limit == false`（已经完全自由）的状态才写进 `dp`。

## 代码

原笔记的带注释版本：

```cpp
class DigitDP {
private:
    vector<vector<int>> dp;
    string digit;
    int len;
    // dp[pos][lead0] 表示已经填了 pos 位，是否有非零数字
    // digit 拆位，全部 - '0'

    // 深度优先搜索
    // pos    : 当前处理到第几位（从 0 开始）
    // limit  : 之前的前缀是否与上界完全一致（即当前位是否受限制）
    // lead0  : 是否已经出现过非零数字（用于处理前导零）
    int dfs(int pos, bool limit, bool lead0) {
        // 递归边界：所有位都已填完
        // 根据题目要求返回 1（该数合法）或 0（不合法）
        // 如果要求数字不能全为零，可以判断 lead0
        if (pos == len) return 1;
        // 记忆化查询：仅当 !limit 时记忆，因为 limit 状态数量少且无需重复
        if (!limit && dp[pos][lead0] != -1) return dp[pos][lead0];
        // 当前位可填的数字上限
        int up = limit ? digit[pos] : 9;
        int res = 0;
        // 枚举当前位所有可能数字
        for (int d = 0; d <= up; ++d) {
            // 根据题目要求添加额外过滤条件

            bool next_limit = limit && (d == up);
            bool next_lead0 = lead0 && (d == 0);
            res += dfs(pos + 1, next_limit, next_lead0);
        }
        // 5. 记忆化存储
        if (!limit) dp[pos][lead0] = res;
        return res;
    }

public:
    explicit DigitDP(int num) {
        digit = to_string(num);
        len = digit.size();
        for (auto &i : digit) i -= '0';
        dp.assign(len, vector<int>(2, -1));
    }

    int count() {
        return dfs(0, true, true);
    }
};

cout << DigitDP(100).count() << endl; // 不添加条件，返回 101，即 [0,100]
```

不加任何过滤条件时，它数出来的就是 $[0, N]$ 的整数个数 $N+1$——这正好是验证模板有没有写错的最快办法。

## 怎么加条件

两条路，按条件是否「依赖历史」区分：

1. **不依赖历史**（只看当前这一位的数字）：直接在 `for` 循环里 `continue`。比如「不含 4」：

```cpp
for (int d = 0; d <= up; ++d) {
    if (d == 4) continue;
    res += dfs(pos + 1, limit && (d == up), lead0 && (d == 0));
}
```

2. **依赖历史**（相邻位、数位和、是否出现过某数字……）：必须把这份历史加进状态，写成 `dp[pos][lead0][last]` 之类，否则记忆化会拿到错答案。

## 区间查询

多数题目要 $[L, R]$ 的答案，直接减：

```cpp
long long ans = 1LL * DigitDP(R).count() - DigitDP(L - 1).count();
```

注意 $L = 0$ 时 `L - 1 = -1`，`to_string(-1)` 会变成 `"-"` 导致 `len = 1` 却拆不出数字，所以要先特判 `L == 0`（或者给 `DigitDP` 加一个「`num < 0` 直接返回 0」的保护）。

## 易错点

- `next_limit = limit && (d == up)`：写成 `d == digit[pos]` 也对，但要注意 `up` 在 unlimited 时是 9，两者语义不同——统一用 `up` 最不容易错。
- 记忆化的条件是 `!limit`，**写反了就错**。
- `dp` 只开了 `pos \times 2` 两维，加了「依赖历史」的条件却不加维度，是这套模板最常见的翻车方式。
- `digit[pos]` 在构造函数里已经减过 `'0'`，别在 `dfs` 里再减一次。
- 值域大、答案要取模时 `int` 会爆，`res` 用 `long long`。

## 对拍验证

- 不加条件：对 $N = 0 \sim 5000$ 逐个检查 `count() == N + 1`，再抽查 `999999`、`1000000`、`123456789`、`2147483647`。
- 加「不含 4」的条件：对 $N = 0 \sim 5000$ 与暴力枚举 $[0, N]$ 逐个比对。

两组都通过。

<!-- 原文：洛谷文章 o7u0laft「DP」 -->
