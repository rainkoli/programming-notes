# Nowcoder Weekly Round 155

## [A. Three Switches and Indicator Light](https://ac.nowcoder.com/acm/contest/138240/A)

## Problem Description

There are three independent switches. Their states are represented by three integers `x1`, `x2`, and `x3`.

For each switch:

- `0` means the switch is off.s
- `1` means the switch is on.

The controller counts how many switches are on:

- If the number of switches that are on is odd, the indicator light is on.
- If the number of switches that are on is even, the indicator light is off.

Output the state of the indicator light.

Constraints:

- `xi ∈ {0, 1}`
- There are exactly three switches.

---

## Example 1

### Input

```text
1 0 0
```

### Output

```text
ON
```

Explanation:

Only one switch is on. Since `1` is odd, the indicator light is on.

---

## Example 2

### Input

```text
1 1 0
```

### Output

```text
OFF
```

Explanation:

Two switches are on. Since `2` is even, the indicator light is off.

---

## How to Figure It Out

We only need to count how many of the three input values are equal to `1`.

Let the number of switches that are on be `count`.

Then:

```text
count % 2 == 1  ->  ON
count % 2 == 0  ->  OFF
```

For example:

```text
Input: 1 0 1
count = 2
2 % 2 = 0
Output: OFF
```

The loop must examine all three elements. Therefore, when an array is used, the loop condition should be:

```java
i < arr.length
```

Using `i < 2` only checks `arr[0]` and `arr[1]`, so `arr[2]` would be ignored.

---

## Java Solution

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);

        int[] arr = new int[3];

        for (int i = 0; i < arr.length; i++) {
            arr[i] = in.nextInt();
        }

        if (calculate(arr)) {
            System.out.println("ON");
        } else {
            System.out.println("OFF");
        }
    }

    public static boolean calculate(int[] arr) {
        int count = 0;

        for (int i = 0; i < arr.length; i++) {
            if (arr[i] == 1) {
                count++;
            }
        }

        return count % 2 == 1;
    }
}
```

---

## Simplified Java Solution

Because there are always exactly three switches, the result can also be calculated directly while reading the input:

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);

        int count = 0;

        for (int i = 0; i < 3; i++) {
            if (in.nextInt() == 1) {
                count++;
            }
        }

        System.out.println(count % 2 == 1 ? "ON" : "OFF");
    }
}
```

---

## Complexity

Because the number of switches is fixed at three:

- Time complexity: `O(1)`
- Space complexity:
  - Array solution: `O(1)`
  - Simplified solution: `O(1)`

Although the array solution contains a loop, it always runs exactly three times, so its time complexity is still constant.

---

## ICPC Classification

The main type is:

- **Implementation**

It also involves:

- Counting
- Parity Checking
- Basic Conditional Statements

It is generally not considered a simulation problem because there is no changing process that must be reproduced step by step. We only count the active switches and check whether the result is odd or even.

**Recommended label:**

> Implementation + Counting + Parity Checking
