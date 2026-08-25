# Nowcoder Weekly Round 152

## A. Xiaohong's Multiple Construction

## Problem Description

Given a positive integer `x`, construct an integer `y` such that:

1. `x < y`
2. `y` is divisible by `x`
3. The set of distinct digits appearing in `y` is exactly the same as the set of distinct digits appearing in `x`

Any valid answer is accepted.

Constraints:

- `1 ≤ x < 10^9`
- `y ≤ 10^18`

---

## Example

### Input

```text
37
```

### One Valid Output

```text
3737
```

Explanation:

```text
3737 = 37 × 101
```

The distinct digits in both `37` and `3737` are `{3, 7}`.

The sample output `7733` is also valid because:

```text
7733 = 37 × 209
```

---

## How to Figure It Out

The problem allows us to output **any valid answer**, so we should try to construct one directly.

Concatenate `x` with itself:

```text
y = xx
```

For example:

```text
x = 37
y = 3737
```

If `x` has `n` digits, concatenating it with itself gives:

$$ y = x \times 10^n + x$$

Factor out `x`:

$$ y = x(10^n + 1)$$

Therefore, `y` is always divisible by `x`.

Because `y` contains two copies of `x`, it has exactly the same distinct digit types as `x`.

Also, since `x < 10^9`, `x` has at most 9 digits, so `y` has at most 18 digits and satisfies `y ≤ 10^18`.

---

## Java Solution

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);

        String x = in.next();
        System.out.println(x + x);
    }
}
```

---

## Complexity

Let `n` be the number of digits in `x`.

- Time complexity: `O(n)`
- Space complexity: `O(n)`

---

## ICPC Classification

The main type is:

- **Constructive Algorithm**

It can also involve:

- Basic Mathematics
- String Manipulation
- Implementation

It is generally **not a simulation problem**, because there is no process that must be reproduced step by step. We directly build a valid answer using a fixed construction rule.

**Recommended label:**

> Constructive Algorithm + Basic Mathematics + String Manipulation
