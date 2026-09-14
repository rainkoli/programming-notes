# LeetCode Game Dynamic Programming

Game DP（博弈 DP）用于处理双方轮流行动、双方均采用最优策略的问题。

## Core Pattern

对于“当前玩家是否必胜”的布尔型博弈 DP：

```text
存在一种操作可以让对手进入必败状态
        ↓
当前状态必胜
```

即：

```text
dp[current] = true
if there exists next such that dp[next] == false
```

典型代码：

```java
if (!dp[next]) {
    dp[current] = true;
    break;
}
```

其中 `break` 是 early exit：一旦找到一种足以证明当前状态必胜的操作，就不需要继续搜索。

对于“最大化当前玩家与对手分差”的博弈 DP，常见形式是：

```text
当前收益 - 对手后续最优收益
```

例如：

```java
dp[i] = Math.max(
    dp[i + 1],
    value[i] - dp[i + 1]
);
```

## Problems

- [1872. Stone Game VIII](./1872-stone-game-viii.md)

## Learning Direction

建议逐步从以下模式学习：

1. Win / Lose 状态 DP
2. 当前玩家视角的 Minimax / Score Difference
3. 区间博弈 DP
4. 多维状态博弈

不要因为出现新的 Game DP 变体就立即创建新的目录；当某一类题目形成稳定的问题族后，再考虑拆分。

Return to [LeetCode Dynamic Programming](../README.md) or [LeetCode / LCR Problems](../../README.md).
