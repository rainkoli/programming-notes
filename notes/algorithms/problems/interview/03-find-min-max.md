# 面试算法题：无序数组中快速找出最大值和最小值

## 1. 题目

给定一个**无序数组**，要求尽可能快地得到：

- 最大值 `max`
- 最小值 `min`

典型问题：

> 一个数组，最快得到最大值和最小值的方法是什么？

这道题的重点有两个：

1. 能不能做到比 `O(n)` 更快；
2. 在时间复杂度都是 `O(n)` 的情况下，能不能减少比较次数。

---

## 2. 最直接的方法：遍历一次

思路：

- 用第一个元素初始化 `min`、`max`；
- 从第二个元素开始遍历；
- 每个元素分别与 `min`、`max` 比较。

### Java 实现

```java
public static int[] findMinMax(int[] nums) {
    if (nums == null || nums.length == 0) {
        throw new IllegalArgumentException("数组不能为空");
    }

    int min = nums[0];
    int max = nums[0];

    for (int i = 1; i < nums.length; i++) {
        if (nums[i] < min) {
            min = nums[i];
        }

        if (nums[i] > max) {
            max = nums[i];
        }
    }

    return new int[]{min, max};
}
```

时间复杂度：

```text
O(n)
```

空间复杂度：

```text
O(1)
```

---

## 3. 普通遍历的最坏比较次数

从第二个元素开始，每个元素最多比较两次：

```text
和 min 比一次
和 max 比一次
```

因此：

\[
2(n-1)=2n-2
\]

例如：

```text
n = 100
```

最多比较：

\[
198
\]

次。

---

## 4. 能不能比 O(n) 更快？

对于没有额外信息的无序数组：

\[
\boxed{\Omega(n)}
\]

原因：

> 如果某个元素从来没有被检查，那么它就可能是最大值或最小值。

所以至少要看一遍所有元素。

因此：

```text
O(n)
```

已经是渐进意义上的最优复杂度。

但是可以进一步优化：

> 减少比较次数。

---

## 5. 两两比较优化

核心思想：

> 两个元素先互相比较，较大的只去挑战 `max`，较小的只去挑战 `min`。

例如：

```text
7, 2
```

先比较：

```text
7 > 2
```

得到：

```text
7 → 只和 max 比
2 → 只和 min 比
```

因此两个元素只需要：

1. 两个元素互相比较一次；
2. 较大的与 `max` 比一次；
3. 较小的与 `min` 比一次。

总共：

```text
3 次比较
```

普通扫描两个元素最多需要：

```text
4 次比较
```

---

## 6. Java 优化实现

```java
public static int[] findMinMax(int[] nums) {
    if (nums == null || nums.length == 0) {
        throw new IllegalArgumentException("数组不能为空");
    }

    int min;
    int max;
    int i;

    if (nums.length % 2 == 0) {
        if (nums[0] < nums[1]) {
            min = nums[0];
            max = nums[1];
        } else {
            min = nums[1];
            max = nums[0];
        }

        i = 2;
    } else {
        min = nums[0];
        max = nums[0];
        i = 1;
    }

    while (i < nums.length - 1) {
        int smaller;
        int larger;

        if (nums[i] < nums[i + 1]) {
            smaller = nums[i];
            larger = nums[i + 1];
        } else {
            smaller = nums[i + 1];
            larger = nums[i];
        }

        if (smaller < min) {
            min = smaller;
        }

        if (larger > max) {
            max = larger;
        }

        i += 2;
    }

    return new int[]{min, max};
}
```

---

## 7. 示例执行

数组：

```text
[7, 2, 9, 4, 1, 8]
```

### 第一组：7 和 2

比较：

```text
7 > 2
```

得到：

```text
max = 7
min = 2
```

比较次数：

```text
1
```

### 第二组：9 和 4

先比较：

```text
9 > 4
```

得到：

```text
larger = 9
smaller = 4
```

再比较：

```text
9 > max(7)
```

成立：

```text
max = 9
```

再比较：

```text
4 < min(2)
```

不成立。

这一组：

```text
3 次比较
```

### 第三组：1 和 8

先比较：

```text
1 < 8
```

得到：

```text
smaller = 1
larger = 8
```

然后：

```text
1 < min(2)
```

成立：

```text
min = 1
```

再比较：

```text
8 > max(9)
```

不成立。

这一组：

```text
3 次比较
```

最终：

```text
min = 1
max = 9
```

总比较次数：

```text
1 + 3 + 3 = 7
```

普通方法最坏需要：

```text
2 × (6 - 1) = 10
```

---

## 8. 比较次数推导

当 `n` 为偶数时：

第一对元素：

```text
1 次比较
```

剩余：

```text
n - 2 个元素
```

共：

\[
\frac{n-2}{2}
\]

组。

每组需要 3 次比较：

\[
1+3\times\frac{n-2}{2}
\]

化简：

\[
1+\frac{3n-6}{2}
\]

\[
=\frac{3n}{2}-2
\]

所以：

\[
\boxed{\frac{3n}{2}-2}
\]

例如 `n = 100`：

普通方法：

\[
2n-2=198
\]

两两比较：

\[
\frac{3n}{2}-2=148
\]

减少：

```text
50 次比较
```

---

## 9. 为什么仍然是 O(n)？

普通遍历：

```text
约 2n 次比较
```

两两比较：

```text
约 1.5n 次比较
```

但：

\[
O(2n)=O(n)
\]

\[
O(1.5n)=O(n)
\]

因此两者时间复杂度都还是：

\[
O(n)
\]

区别只是：

> 两两比较优化了常数项。

所以：

```text
时间复杂度相同
≠
实际比较次数相同
```

---

## 10. 为什么不应该先排序？

如果只想知道最大值和最小值，没有必要先把整个数组排序。

常见排序通常需要：

\[
O(n\log n)
\]

排序后再取：

```java
int min = nums[0];
int max = nums[nums.length - 1];
```

属于获取了远远超过需求的信息。

类比：

> 只想知道全班第一名和最后一名，却先把所有人的完整排名排出来。

因此：

```text
只求 max/min → 不需要排序
```

---

## 11. 与 Top K 的区别

聊天里还提到了：

```text
是不是问最大的几个数？
最小堆？
Top K？
```

如果题目变成：

> 在 n 个数中找最大的 k 个数。

这就是 Top K。

常用方法：

```text
维护大小为 k 的最小堆
```

时间复杂度：

\[
O(n\log k)
\]

空间复杂度：

\[
O(k)
\]

所以要区分：

### 最大值 + 最小值

```text
成对比较
O(n)
O(1)
```

### 最大的 K 个

```text
最小堆
O(n log k)
O(k)
```

---

## 12. 面试推荐回答

可以回答：

> 对于无序数组，至少必须检查每个元素一次，所以时间复杂度下界是 Ω(n)，一次遍历 O(n) 已经是渐进最优。
>
> 如果还要优化比较次数，可以把元素两两分组，每组先内部比较一次，较大的只与当前最大值比较，较小的只与当前最小值比较。
>
> 这样时间复杂度仍然是 O(n)，额外空间 O(1)，但偶数 n 时最坏比较次数可以从 `2n - 2` 降到 `3n / 2 - 2`。

---

## 13. 核心总结

| 项目 | 普通遍历 | 两两比较 |
|---|---:|---:|
| 时间复杂度 | O(n) | O(n) |
| 空间复杂度 | O(1) | O(1) |
| 最坏比较次数（偶数 n） | 2n - 2 | 3n / 2 - 2 |
| 是否排序 | 否 | 否 |
| 优化点 | 简单 | 比较次数更少 |

记忆：

```text
1. 无序数组求 max/min 的复杂度下界是 Ω(n)
2. 不需要排序
3. 普通遍历最多 2n - 2 次比较
4. 两两比较可以降到 3n/2 - 2
5. Top K 是另一类问题，通常使用最小堆
```
