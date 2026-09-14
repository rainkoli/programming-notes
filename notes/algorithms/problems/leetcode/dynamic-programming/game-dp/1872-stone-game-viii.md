# 1872 - Stone Game VIII

> LeetCode 1872 · Dynamic Programming · Game DP · Prefix Sum · Backward DP

---

## 1. Problem

Alice 和 Bob 轮流进行游戏，Alice 先手。

给定：

```text
stones = [a0, a1, a2, ..., a(n-1)]
```

每次操作：

1. 选择一个整数 `x > 1`。
2. 移除最左边的 `x` 个石子。
3. 把这 `x` 个石子的和加入当前玩家分数。
4. 把这个和作为一个新石子放回最左边。
5. 当只剩一个石子时游戏结束。

题目最终要求：

```text
Alice score - Bob score
```

Alice 希望这个值尽可能大，Bob 希望这个值尽可能小。

双方都采用最优策略。

---

# 2. 最终代码

```java
class Solution {
    public int stoneGameVIII(int[] stones) {
        int n = stones.length;

        for (int i = 1; i < n; i++) {
            stones[i] += stones[i - 1];
        }

        int[] dp = new int[n];

        dp[n - 1] = stones[n - 1];

        for (int i = n - 2; i > 0; i--) {
            dp[i] = Math.max(
                    dp[i + 1],
                    stones[i] - dp[i + 1]
            );
        }

        return dp[1];
    }
}
```

---

# 3. 这道题真正困难的地方

代码并不长，真正困难的是理解为什么：

```java
dp[i] = Math.max(
    dp[i + 1],
    stones[i] - dp[i + 1]
);
```

成立。

理解链条应该是：

```text
原始石子游戏
    ↓
每一次合并产生的值其实都是原数组的某个前缀和
    ↓
把游戏转换成“选择 prefix 下标”
    ↓
每次选过一个 prefix[i] 后，
下一位玩家只能选择更大的 prefix 下标
    ↓
用 i 表示“当前最小可选 prefix 下标”
    ↓
定义 dp[i] 为这个局面的价值
    ↓
当前玩家有两类决策：
不选 i / 选择 i
    ↓
得到状态转移方程
```

---

# 4. 第一步：为什么要做前缀和

原代码：

```java
for (int i = 1; i < n; i++) {
    stones[i] += stones[i - 1];
}
```

执行以后：

```text
stones[i]
```

不再表示原来的第 `i` 个石子，而表示：

```text
prefix[i]
= 原数组 stones[0] + ... + stones[i]
```

即：

```text
prefix[0] = 前 1 个石子的和
prefix[1] = 前 2 个石子的和
prefix[2] = 前 3 个石子的和
...
prefix[i] = 前 i + 1 个石子的和
```

## 为什么原游戏能转换成前缀和？

假设原数组：

```text
[a, b, c, d, e]
```

Alice 先取前两个：

```text
a + b
```

数组变成：

```text
[a+b, c, d, e]
```

Bob 如果接着取当前前两个：

```text
(a+b) + c
= a+b+c
= prefix[2]
```

Bob 如果取当前前三个：

```text
(a+b) + c + d
= a+b+c+d
= prefix[3]
```

所以无论中间经历过多少次合并：

> 每次操作产生的新石子，本质上都对应原数组的某个前缀和。

因此原游戏可以被重新理解为：

```text
不断选择 prefix[i]
```

并且选择下标只能越来越大。

---

# 5. `x` 与 `prefix[i]` 的下标关系

这是本题最容易弄混的地方之一。

```text
prefix[i]
```

表示前：

```text
i + 1
```

个石子的和。

所以：

```text
x = 1  -> prefix[0]   ❌ 题目不允许
x = 2  -> prefix[1]   ✅
x = 3  -> prefix[2]   ✅
x = 4  -> prefix[3]   ✅
x = 5  -> prefix[4]   ✅
```

记住：

```text
选择 x 个石子
↕
选择 prefix[x - 1]
```

因此：

> `prefix[3]` 不是前 3 个石子的和，而是前 4 个石子的和。

---

# 6. 状态 `i` 到底表示什么

在这个 DP 中：

```text
i
```

不是“已经选择了 prefix[i]”。

而是：

> 当前最小允许选择的 prefix 下标。

例如：

```text
dp[2]
```

对应候选范围：

```text
prefix[2], prefix[3], ..., prefix[n - 1]
```

所以可以把 `i` 看成：

```text
候选区间的左边界
```

即：

```text
dp[i]

候选：
[i, i+1, i+2, ..., n-1]
 ↑
 最小可选下标
```

---

# 7. `dp[i]` 的精确定义

## 推荐定义

```text
dp[i]
=
当前行动方最早可以从 prefix[i] 开始选择时，
在双方都采用最优策略的情况下，
当前行动方最终能够取得的最大
“当前行动方得分 - 对手得分”。
```

注意两个关键点。

### 7.1 `dp[i]` 不绑定 Alice 或 Bob

`dp[i]` 不是：

```text
Alice 的 dp
```

也不是：

```text
Bob 的 dp
```

而是：

```text
谁现在拥有行动权，
dp[i] 就站在谁的视角。
```

所以：

```text
如果当前玩家是 Alice：
dp[i] = Alice - Bob 的最大值

如果当前玩家是 Bob：
dp[i] = Bob - Alice 的最大值
```

这是一种“当前玩家视角”。

---

### 7.2 `dp[i]` 可以理解成“场面价值”

一个更抽象的理解是：

```text
dp[i]
=
最小可选下标为 i 的这个游戏局面的价值
```

它并不是在模拟：

```text
Alice -> Bob -> Alice -> Bob
```

而是在给一个“局面”预先求最优值。

因此：

```text
dp[1]
dp[2]
dp[3]
...
```

可以看成一张：

```text
局面价值表
```

只是这个价值始终以“当前行动方”的视角归一化。

---

# 8. 为什么 Bob 也可以使用 `Math.max`

题目原始定义：

```text
D = Alice - Bob
```

Alice 希望：

```text
max(D)
```

Bob 希望：

```text
min(D)
```

看起来应该是：

```text
Alice -> max
Bob   -> min
```

但是：

```text
Bob - Alice = -(Alice - Bob)
```

所以 Bob 希望：

```text
min(Alice - Bob)
```

完全等价于：

```text
max(Bob - Alice)
```

因此如果统一定义为：

```text
当前玩家 - 对手
```

那么：

```text
Alice 回合：
max(Alice - Bob)

Bob 回合：
max(Bob - Alice)
```

所以双方都可以统一为：

```java
Math.max(...)
```

这就是为什么只需要一个 `dp[]`。

---

# 9. Base Case 是什么

## 9.1 一般定义

`base case`：

> 一个可以直接确定答案、不需要继续依赖其他 DP 状态的最基础状态。

本题：

```java
dp[n - 1] = stones[n - 1];
```

就是 base case。

---

## 9.2 为什么 `dp[n - 1] = stones[n - 1]`

状态：

```text
dp[n - 1]
```

表示最小可选位置已经是：

```text
prefix[n - 1]
```

但它已经是最后一个 prefix 了。

候选集合只有：

```text
{ prefix[n - 1] }
```

没有：

```text
prefix[n]
prefix[n + 1]
...
```

所以当前玩家没有选择空间，只能选择：

```text
prefix[n - 1]
```

选择后所有石子被合并成一个新石子，游戏立即结束。

所以：

```text
当前玩家得分 = prefix[n - 1]
对手后续得分 = 0
```

因此：

```text
当前玩家 - 对手
= prefix[n - 1]
```

又因为前缀和已经原地写回 `stones`：

```text
stones[n - 1] = prefix[n - 1]
```

所以：

```java
dp[n - 1] = stones[n - 1];
```

成立。

---

# 10. Recurrence 是什么

`recurrence`：

> 已知较小子问题的答案以后，如何推导当前问题的答案。

本题的 recurrence / 状态转移方程：

```java
dp[i] = Math.max(
    dp[i + 1],
    stones[i] - dp[i + 1]
);
```

它表示：

```text
当前状态 dp[i]
有两大类选择：

A. 不选择 prefix[i]
B. 就选择 prefix[i]
```

---

# 11. 第一项：`dp[i + 1]` 的意义

```java
dp[i + 1]
```

表示：

> 当前玩家不选择 `prefix[i]`，把这个候选排除掉，继续从 `prefix[i+1]` 以及更后面的候选中寻找最优答案。

注意：

> “跳过 i”不是跳过自己的游戏回合。

并不存在：

```text
Alice：我不操作了
Bob：那换我操作
```

而是：

```text
当前玩家仍然在进行本次选择

只是不选：
prefix[i]

而改为从：
prefix[i + 1], prefix[i + 2], ...
中选择
```

为什么可以直接写：

```text
dp[i + 1]
```

？

因为：

```text
从 prefix[i + 1] 开始的整个子问题
```

已经被 `dp[i + 1]` 预先计算好了。

所以：

```text
dp[i + 1]
=
“不选择当前 i，去后面找最优”的结果
```

---

# 12. 第二项：`stones[i] - dp[i + 1]` 的意义

这是本题最重要的式子。

```java
stones[i] - dp[i + 1]
```

表示：

> 当前玩家真的选择 `prefix[i]` 以后，这条选择最终产生的净分差。

拆开：

```text
stones[i]
```

表示：

> 当前玩家这一手立即获得的分数。

而选择 `prefix[i]` 后，游戏真的执行了一次操作，于是：

```text
轮到对手
```

对手接下来面对的最小可选状态就是：

```text
i + 1
```

因此：

```text
dp[i + 1]
```

此时是：

> 对手在后续局面能够取得的最大“对手 - 当前玩家”分差。

因此当前玩家自己的最终优势：

```text
当前玩家当前得到的分数
-
对手后续的最优优势
```

即：

```java
stones[i] - dp[i + 1]
```

---

# 13. 为什么是减法，不是加法

从纯 DP 的角度，可以理解成：

```text
选择当前动作的方案价值
=
当前动作的即时价值
+
后续子状态经过状态关系转换后的价值
```

这里：

```text
当前即时价值 = stones[i]
```

而：

```text
后续状态 dp[i + 1]
```

是从“下一位行动方”的视角定义的。

所以要把它转换回当前玩家视角：

```text
-dp[i + 1]
```

因此：

```text
stones[i] + (-dp[i + 1])
=
stones[i] - dp[i + 1]
```

和普通 DP 一样：

```text
当前状态
=
当前贡献
+
已经解决的子问题贡献
```

只是本题的后续贡献因为视角相反，所以要取负。

---

# 14. 为什么要 `Math.max`

```java
dp[i] = Math.max(
    dp[i + 1],
    stones[i] - dp[i + 1]
);
```

两项分别是：

```text
方案 A：
不选当前 prefix[i]
→ dp[i + 1]

方案 B：
选择当前 prefix[i]
→ stones[i] - dp[i + 1]
```

`dp[i]` 定义的是：

```text
当前玩家能够取得的最大分差
```

因此当然需要：

```java
Math.max(...)
```

---

# 15. 为什么从右往左计算

状态转移：

```text
dp[i]
依赖
dp[i + 1]
```

所以计算 `dp[i]` 以前，必须保证：

```text
dp[i + 1]
```

已经计算完成。

因此：

```text
dp[n - 1]
↓
dp[n - 2]
↓
dp[n - 3]
↓
...
↓
dp[1]
```

对应代码：

```java
for (int i = n - 2; i > 0; i--) {
    ...
}
```

---

# 16. 为什么循环从 `n - 2` 开始

因为：

```text
dp[n - 1]
```

已经作为 base case 直接计算好了：

```java
dp[n - 1] = stones[n - 1];
```

所以下一个需要计算的是：

```text
dp[n - 2]
```

因此：

```java
int i = n - 2;
```

---

# 17. 为什么循环只算到 `i = 1`

题目规定：

```text
x > 1
```

意味着至少选择：

```text
2 个石子
```

而：

```text
前 2 个石子的和 = prefix[1]
```

所以：

```text
最小合法 prefix 下标 = 1
```

因此不需要计算：

```text
dp[0]
```

因为 `dp[0]` 相当于允许选择前 1 个石子，不符合题意。

---

# 18. 为什么最后 `return dp[1]`

题目有两个条件：

```text
Alice 先手
x > 1
```

所以游戏初始时：

```text
当前行动方 = Alice
```

最小合法候选：

```text
prefix[1]
```

因此游戏的初始状态就是：

```text
dp[1]
```

而 `dp[1]` 在 Alice 是当前玩家时表示：

```text
Alice - Bob
```

的最大最终分差。

因此：

```java
return dp[1];
```

正好返回题目要求的答案。

---

# 19. Case：`[-1, 2, -3, 4, -5]`

输入：

```text
stones = [-1, 2, -3, 4, -5]
```

预期输出：

```text
5
```

---

## 19.1 前缀和

```text
index:       0    1    2    3    4
原数组:     -1    2   -3    4   -5
```

依次执行：

```text
i = 1
stones[1] = 2 + (-1) = 1

i = 2
stones[2] = -3 + 1 = -2

i = 3
stones[3] = 4 + (-2) = 2

i = 4
stones[4] = -5 + 2 = -3
```

最终：

```text
index:       0    1    2    3    4
prefix:     -1    1   -2    2   -3
```

---

# 20. 代码时间线 —— 重点

这是手推代码时最应该看的时间线。

> 代码为了计算当前状态，先把它依赖的“未来状态”算出来。

初始化：

```text
dp = [0, 0, 0, 0, 0]
```

base case：

```text
dp[4] = stones[4] = -3

dp = [0, 0, 0, 0, -3]
```

---

## i = 3

```text
已知：

stones[3] = 2
dp[4] = -3
```

不选 3：

```text
dp[4] = -3
```

选择 3：

```text
stones[3] - dp[4]
= 2 - (-3)
= 5
```

所以：

```text
dp[3]
= max(-3, 5)
= 5
```

数组：

```text
dp = [0, 0, 0, 5, -3]
```

---

## i = 2

```text
已知：

stones[2] = -2
dp[3] = 5
```

不选 2：

```text
dp[3] = 5
```

选择 2：

```text
stones[2] - dp[3]
= -2 - 5
= -7
```

所以：

```text
dp[2]
= max(5, -7)
= 5
```

数组：

```text
dp = [0, 0, 5, 5, -3]
```

---

## i = 1

```text
已知：

stones[1] = 1
dp[2] = 5
```

不选 1：

```text
dp[2] = 5
```

选择 1：

```text
stones[1] - dp[2]
= 1 - 5
= -4
```

所以：

```text
dp[1]
= max(5, -4)
= 5
```

最终：

```text
dp = [0, 5, 5, 5, -3]
```

返回：

```text
dp[1] = 5
```

---

# 21. 代码时间线总图 —— 必背

```text
prefix:

index       0    1    2    3    4
value      -1    1   -2    2   -3


先算最右边：

dp[4] = -3


然后：

i = 3

不选3:
dp[4] = -3

选3:
stones[3] - dp[4]
= 2 - (-3)
= 5

dp[3] = 5


然后：

i = 2

不选2:
dp[3] = 5

选2:
stones[2] - dp[3]
= -2 - 5
= -7

dp[2] = 5


然后：

i = 1

不选1:
dp[2] = 5

选1:
stones[1] - dp[2]
= 1 - 5
= -4

dp[1] = 5


answer = 5
```

程序计算方向：

```text
dp[4] -> dp[3] -> dp[2] -> dp[1]

未来答案
    ↓
推导当前答案
```

---

# 22. 游戏时间线 —— 重点

这是理解：

```text
为什么 dp[i + 1] 是“未来局面”
```

以及：

```text
为什么 stones[i] - dp[i + 1]
```

成立的关键。

考虑最优路径。

原数组：

```text
[-1, 2, -3, 4, -5]
```

Alice 先手。

Alice 选择：

```text
x = 4
```

也就是选择：

```text
prefix[3]
```

前 4 个石子的和：

```text
-1 + 2 - 3 + 4 = 2
```

所以：

```text
Alice score = 2
```

数组变成：

```text
[2, -5]
```

现在轮到 Bob。

Bob 面前只有两个石子，且规则要求：

```text
x > 1
```

所以 Bob 只能：

```text
x = 2
```

Bob 得分：

```text
2 + (-5) = -3
```

数组：

```text
[-3]
```

只剩一个石子，游戏结束。

最终：

```text
Alice = 2
Bob   = -3
```

所以：

```text
Alice - Bob
= 2 - (-3)
= 5
```

---

# 23. 游戏时间线与 DP 状态对应

Alice 处于：

```text
dp[3]
```

如果 Alice 选择：

```text
prefix[3]
```

那么：

```text
Alice 当前得到：
stones[3] = 2
```

然后游戏进入：

```text
dp[4]
```

此时当前玩家变成 Bob。

所以：

```text
dp[4] = -3
```

在这个具体分支里，可以理解成：

```text
Bob 在后续局面的最优分差 = -3
```

于是 Alice 这条选择最终的价值：

```text
stones[3] - dp[4]
= 2 - (-3)
= 5
```

也就是：

```text
Alice - Bob = 5
```

---

# 24. 为什么说 `dp[4]` 是 `dp[3]` 的“未来局面”

真实游戏：

```text
dp[3]
  ↓
当前玩家选择 prefix[3]
  ↓
执行一次真实操作
  ↓
游戏进入 dp[4]
```

所以游戏时间方向是：

```text
dp[3] -> dp[4]

现在       未来
```

但是代码为了计算：

```text
dp[3]
```

必须先知道：

```text
dp[4]
```

所以代码方向反过来：

```text
dp[4] -> dp[3]

先算未来     再算现在
```

---

# 25. 两条时间线必须区分

## 游戏时间线

```text
现在 -> 未来

dp[1] -> dp[2] -> dp[3] -> dp[4]
```

表示：

```text
执行当前动作后，
真实游戏可能进入更后的状态。
```

---

## 代码计算时间线

```text
未来答案 -> 当前答案

dp[4] -> dp[3] -> dp[2] -> dp[1]
```

表示：

```text
先把最远的未来状态算好，
再利用它倒推当前状态。
```

---

## 最核心的一句话

```text
游戏是从“现在”走向“未来”。

DP 为了决定“现在怎么选”，
反过来先把“未来会怎样”算出来。
```

这就是本题从右向左 DP 的本质。

---

# 26. 为什么两条时间线不能混在一起

例如：

```text
dp[4]
```

在代码上：

```text
它比 dp[3] 先计算。
```

但是在游戏里：

```text
如果当前处于 dp[3]，
选择 prefix[3] 后，
才可能进入 dp[4]。
```

所以不能说：

```text
dp[4] 是游戏中的“上一轮”
```

正确区分：

```text
代码时间：
dp[4] 是“之前已经算好的子问题”

游戏时间：
dp[4] 是“当前动作之后进入的未来局面”
```

---

# 27. `dp[3] = max(dp[4], stones[3] - dp[4])` 的完整解释

本 case：

```text
prefix:

index       3      4
value       2     -3
```

`dp[3]` 表示当前可以选择：

```text
prefix[3]
prefix[4]
```

两种路线：

```text
                      dp[3]
                        |
          +-------------+-------------+
          |                           |
    不选 prefix[3]               选 prefix[3]
          |                           |
   从 prefix[4] 开始               取前 4 个
          |                           |
      value = dp[4]              当前得到 2
          |                           |
         -3                        轮到对手
                                      |
                                   进入 dp[4]
                                      |
                            对手后续最优差 = -3
                                      |
                           当前最终差 = 2 - (-3)
                                      |
                                      5
          |                           |
          +-------------+-------------+
                        |
                       max
                        |
                    dp[3] = 5
```

这里：

```text
dp[4]
```

因为已经是最后一个候选，所以恰好对应：

```text
直接选择前 5 个石子
```

但一般情况下：

```text
dp[i + 1]
```

不是简单等于：

```text
选择 prefix[i + 1]
```

而是：

```text
从 prefix[i + 1] 以及更后面的所有选择中得到的最优结果
```

---

# 28. 和普通 Dynamic Programming 的关系

抛开博弈背景：

```java
dp[i] = Math.max(
    dp[i + 1],
    stones[i] - dp[i + 1]
);
```

就是一个标准状态转移方程。

它可以理解为：

```text
当前状态最优值
=
max(
    不采取当前动作的方案价值,
    采取当前动作的方案价值
)
```

即：

```text
dp[i]
=
max(
    skipValue,
    takeValue
)
```

其中：

```text
skipValue = dp[i + 1]
```

```text
takeValue = stones[i] - dp[i + 1]
```

和打家劫舍类似：

```text
dp[i]
=
max(
    不选当前,
    选择当前
)
```

区别只是具体的状态定义和状态关系不同。

---

# 29. 与打家劫舍类 DP 的共性

打家劫舍：

```java
dp[i] = Math.max(
    dp[i - 1],
    dp[i - 2] + nums[i]
);
```

含义：

```text
不抢当前
vs
抢当前
```

Stone Game VIII：

```java
dp[i] = Math.max(
    dp[i + 1],
    stones[i] - dp[i + 1]
);
```

含义：

```text
不选当前 prefix
vs
选当前 prefix
```

共同套路：

```text
1. 定义状态
2. 找 base case
3. 找当前状态有哪些决策
4. 用已知子状态计算每个决策的价值
5. 取最优
6. 根据依赖方向确定遍历顺序
```

---

# 30. 本题 DP 五要素总结

## 30.1 状态

```text
dp[i]
=
当前行动方最早可以从 prefix[i] 开始选择时，
双方最优情况下，
当前行动方能够取得的最大净分差。
```

## 30.2 Base Case

```java
dp[n - 1] = stones[n - 1];
```

## 30.3 决策

```text
不选 prefix[i]

或

选择 prefix[i]
```

## 30.4 Recurrence

```java
dp[i] = Math.max(
    dp[i + 1],
    stones[i] - dp[i + 1]
);
```

## 30.5 遍历方向

```text
dp[i] 依赖 dp[i + 1]
```

所以：

```text
从右向左
```

---

# 31. 题目条件与代码逐项匹配

| 题目条件 | 代码 / DP 中的对应 |
|---|---|
| 一共有 `n` 个石子 | `int n = stones.length` |
| 每次只能取最左边的一段 | 前缀和 |
| 取出的石子求和 | `stones[i]` 被转换成 prefix |
| 新石子值等于这次求和 | 后续仍可表示为更大的原数组前缀 |
| `x > 1` | 最小合法 prefix 下标为 `1` |
| Alice 先手 | 初始 `dp[1]` 的当前行动方就是 Alice |
| Alice 最大化 `Alice-Bob` | 最大化“当前玩家-对手” |
| Bob 最小化 `Alice-Bob` | 等价于最大化 `Bob-Alice` |
| 双方都最优 | `Math.max(...)` |
| 真实执行一次操作后换人 | `stones[i] - dp[i+1]` |
| 不选择当前 prefix | `dp[i+1]` |
| 只剩一个石子结束 | `dp[n-1] = stones[n-1]` |
| 返回 Alice-Bob | `return dp[1]` |

---

# 32. 手推时应该主要看哪条时间线

## 写代码 / 手推数组时

优先看：

```text
代码计算时间线
```

即：

```text
dp[n - 1]
↓
dp[n - 2]
↓
...
↓
dp[1]
```

因为这告诉我们：

```text
dp[i + 1] 已经算好
↓
使用它计算 dp[i]
```

---

## 证明公式为什么成立时

才看：

```text
游戏时间线
```

用来解释：

```text
为什么选择 prefix[i] 后，
下一状态是 dp[i + 1]

以及

为什么要用：
stones[i] - dp[i + 1]
```

一旦状态转移公式已经理解：

> 后续实际手推代码时，不需要每一轮都重新模拟 Alice / Bob。

---

# 33. 常见误区

## 误区 1：`prefix[3]` 表示前 3 个

错误。

```text
prefix[3] = 前 4 个石子的和
```

---

## 误区 2：`dp[i]` 表示已经选择了 `prefix[i]`

错误。

```text
i = 当前最小可选 prefix 下标
```

`dp[i]` 表示从这个候选范围开始的最优场面价值。

---

## 误区 3：`dp[i + 1]` 表示“换 Bob 操作”

不一定。

在：

```text
dp[i + 1]
```

这个 `Math.max` 的第一个分支中：

```text
当前玩家只是排除 prefix[i]
```

没有真正执行游戏操作，所以没有换人。

只有在：

```text
stones[i] - dp[i + 1]
```

这个分支中真的选择了 `prefix[i]`，才发生一次真实操作并换人。

---

## 误区 4：代码先算 `dp[4]`，所以游戏也先发生 `dp[4]`

错误。

```text
代码计算：
dp[4] -> dp[3]

游戏：
dp[3] -> dp[4]
```

这是两条相反的时间线。

---

## 误区 5：`stones[i]` 是差值

错误。

前缀和处理后：

```text
stones[i]
```

只是：

```text
选择 prefix[i] 时，
当前玩家这一手立即得到的分数。
```

它本身不是 Alice-Bob 差值。

---

## 误区 6：`dp[i + 1]` 永远等于选择 `prefix[i + 1]`

错误。

正确：

```text
dp[i + 1]
=
从 prefix[i + 1] 及更后面的所有合法候选中得到的最优结果。
```

只有在：

```text
i + 1 = n - 1
```

时，因为只剩最后一个候选：

```text
dp[n - 1] = prefix[n - 1]
```

---

# 34. 一页复习版

```text
Stone Game VIII

1. 原游戏每次合并得到的值都是原数组某个 prefix。

2. prefix[i] 表示前 i+1 个石子的和。

3. x > 1
   → 最早合法 prefix = prefix[1]
   → 最终答案从 dp[1] 开始。

4. dp[i]：
   当前行动方最早可以从 prefix[i] 开始选时，
   双方最优情况下，
   当前行动方能取得的最大
   “当前行动方 - 对手”净分差。

5. base case：

   dp[n-1] = stones[n-1]

   因为最后只剩 prefix[n-1] 一个候选，
   当前玩家只能选它，随后游戏结束。

6. recurrence：

   dp[i] = max(
       dp[i+1],
       stones[i] - dp[i+1]
   )

   dp[i+1]
   = 不选当前 prefix[i]，
     从后面候选继续寻找最优。

   stones[i] - dp[i+1]
   = 当前真的选择 prefix[i]，
     当前得到 stones[i]，
     然后后续子状态属于下一位行动方，
     因此将其取反后组合回当前玩家视角。

7. dp[i] 依赖 dp[i+1]
   → 从右向左计算。

8. 两条时间线：

   游戏：
   现在 -> 未来

   dp[3] -> dp[4]

   代码：
   未来答案 -> 当前答案

   dp[4] -> dp[3]

9. Case：

   原数组：
   [-1, 2, -3, 4, -5]

   prefix：
   [-1, 1, -2, 2, -3]

   dp[4] = -3

   dp[3]
   = max(-3, 2 - (-3))
   = 5

   dp[2]
   = max(5, -2 - 5)
   = 5

   dp[1]
   = max(5, 1 - 5)
   = 5

   return dp[1] = 5
```

---

# 35. 本题理解 Loop

以后复习这题，不直接背公式，按下面顺序重新推一遍。

## Phase 1：原始操作

问自己：

```text
每次取最左边 x 个以后，
新石子的值是什么？
```

回答：

```text
被移除石子的和
```

---

## Phase 2：发现前缀和

验证：

```text
经过多轮合并后，
新石子的值仍然对应原数组某个 prefix。
```

---

## Phase 3：下标关系

确认：

```text
prefix[i]
= 前 i+1 个石子的和

x > 1
→ 最早合法 i = 1
```

---

## Phase 4：状态定义

自己说出：

```text
dp[i]
=
最小可选 prefix 下标为 i 时，
当前行动方的最大净优势。
```

---

## Phase 5：Base Case

自己证明：

```text
dp[n-1] = stones[n-1]
```

原因：

```text
只剩一个候选，
没有决策空间。
```

---

## Phase 6：两个决策

当前：

```text
dp[i]
```

问：

```text
不选 i 怎么办？
```

答案：

```text
dp[i+1]
```

再问：

```text
真的选 i 怎么办？
```

答案：

```text
stones[i] - dp[i+1]
```

---

## Phase 7：写出 recurrence

```java
dp[i] = Math.max(
    dp[i + 1],
    stones[i] - dp[i + 1]
);
```

---

## Phase 8：两条时间线

一定要分别说出：

```text
游戏：
dp[i] -> dp[i+1]

代码：
dp[i+1] -> dp[i]
```

并解释：

```text
游戏从现在走向未来，
DP 从已经求好的未来答案倒推现在。
```

---

## Phase 9：完整手推 case

必须能独立推出：

```text
prefix = [-1, 1, -2, 2, -3]

dp[4] = -3
dp[3] = 5
dp[2] = 5
dp[1] = 5
```

---

## Phase 10：脱离题解重写代码

只给自己：

```java
class Solution {
    public int stoneGameVIII(int[] stones) {

    }
}
```

然后依次写：

```text
1. n
2. 前缀和
3. dp[]
4. base case
5. 从右向左 recurrence
6. return dp[1]
```

如果某一步写不出来，就回到对应 Phase，而不是直接背代码。

---

# 36. 最终理解

这道题可以抽象成：

```text
复杂游戏
↓
前缀和转换
↓
把游戏历史压缩成一个状态 i
↓
给每个 state i 计算一个局面价值 dp[i]
↓
从最远未来局面开始
↓
利用已求出的未来状态
倒推当前状态
↓
得到初始局面 dp[1]
```

最重要的两句话：

> `dp[i]` 不是 Alice 或 Bob 专属的数据，而是“当前行动方处于 state i 时，这个局面的最优价值”。

> 游戏时间是“现在 → 未来”，而 DP 代码计算时间是“未来答案 → 当前答案”。

---

## Tags

```text
#dynamic-programming
#game-dp
#prefix-sum
#backward-dp
#state-transition
#leetcode-1872
```
