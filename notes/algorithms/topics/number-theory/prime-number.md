The best implementation depends on the range of the input:

- For ordinary Java `int` values, trial division up to (\sqrt n) is simple and reliable.
- For Java `long` values, deterministic Miller–Rabin is much faster.
- For checking many numbers in a range, use the Sieve of Eratosthenes.

# 1. Definition of a prime number

A positive integer (n) is prime when:

$$n>1$$ 

and its only positive divisors are:

$$1 \quad\text{and}\quad n$$

Therefore:

- (0) is not prime.
- (1) is not prime.
- Negative numbers are not prime.
- (2) is prime.

------

# Version 1: Test every possible divisor

```java
public static boolean isPrimeV1(int n) {
    if (n < 2) {
        return false;
    }

    for (int divisor = 2; divisor < n; divisor++) {
        if (n % divisor == 0) {
            return false;
        }
    }

    return true;
}
```

For example, to determine whether (7) is prime, this method tests:

$$7 \bmod 2,\quad
7 \bmod 3,\quad
7 \bmod 4,\quad
7 \bmod 5,\quad
7 \bmod 6$$

None of them equals zero, so (7) is prime.

For (9):

$$9 \bmod 3 = 0$$

Therefore, (9) is not prime.

## Correctness proof

Suppose the method returns `true`.

That means there is no integer (d) satisfying:

$$2 \le d < n$$

such that:

$$n \bmod d = 0$$

Therefore, the only positive divisors of (n) are (1) and (n), so (n) is prime.

Suppose the method returns `false`.

Then it found a divisor (d) for which:

$$2 \le d < n$$ 

and:

$$n \bmod d = 0$$

Thus, (n) has a divisor other than (1) and itself, so it is not prime.

## Complexity

The loop may run approximately (n) times.

$$T(n)=O(n)$$

Space complexity:

$$O(1)$$

This implementation is correct, but unnecessarily slow.

------

# Version 2: Test only up to (n/2)

```java
public static boolean isPrimeV2(int n) {
    if (n < 2) {
        return false;
    }

    for (int divisor = 2; divisor <= n / 2; divisor++) {
        if (n % divisor == 0) {
            return false;
        }
    }

    return true;
}
```

## Why can we stop at (n/2)?

Suppose (d) is a proper divisor of (n), meaning:

$$1 < d < n$$

If:

$$d>\frac n2$$

then multiplying (d) by any integer greater than or equal to (2) gives:

$$2d>n$$

Therefore, (d) cannot divide (n), because the first multiple of (d) after (d) is already larger than (n).

Consequently, no proper divisor of (n) can be greater than (n/2).

## Complexity

$$
O(n)
$$

Although it checks about half as many numbers as Version 1, its asymptotic complexity remains (O(n)).

We can make a much more important improvement.

------

# Version 3: Test only up to (\sqrt n)

```java
public static boolean isPrimeV3(int n) {
    if (n < 2) {
        return false;
    }

    for (int divisor = 2; divisor <= Math.sqrt(n); divisor++) {
        if (n % divisor == 0) {
            return false;
        }
    }

    return true;
}
```

## Why is checking up to (\sqrt n) sufficient?

Suppose (n) is composite. Then there are integers (a) and (b) such that:

$$
n=a\times b
$$

where:

$$
1<a<n,\qquad 1<b<n
$$

Assume both factors were greater than (\sqrt n):

$$
a>\sqrt n
$$

and:

$$
b>\sqrt n
$$

Multiplying these inequalities gives:

$$
ab>\sqrt n\times\sqrt n
$$

Therefore:

$$
ab>n
$$

But we know:

$$
ab=n
$$

This is a contradiction.

Therefore, at least one factor must satisfy:

$$
a\le\sqrt n
$$

or:

$$
b\le\sqrt n
$$

So if (n) is composite, it must have at least one divisor not greater than (\sqrt n).

This means that when no divisor is found between (2) and (\sqrt n), (n) must be prime.

## Example: (36)

The factor pairs are:

$$
1\times36
$$

$$
2\times18
$$

$$
3\times12
$$

$$
4\times9
$$

$$
6\times6
$$

After (6=\sqrt{36}), the pairs simply appear in reverse:

$$
9\times4,\quad12\times3,\quad18\times2,\quad36\times1
$$

Therefore, testing factors after (\sqrt n) provides no new information.

## Complexity

The loop runs approximately (\sqrt n) times:

$$
T(n)=O(\sqrt n)
$$

Space complexity:

$$
O(1)
$$

This is much better than (O(n)).

However, there is a minor implementation issue: `Math.sqrt` uses floating-point arithmetic and is evaluated during every loop condition.

------

# Version 4: Avoid `Math.sqrt`

```java
public static boolean isPrimeV4(int n) {
    if (n < 2) {
        return false;
    }

    for (int divisor = 2; divisor <= n / divisor; divisor++) {
        if (n % divisor == 0) {
            return false;
        }
    }

    return true;
}
```

You might normally write:

```java
divisor * divisor <= n
```

But multiplication can overflow for sufficiently large integer values.

For example, if `divisor` is large, this expression:

```java
divisor * divisor
```

might exceed `Integer.MAX_VALUE` and become a negative or incorrect value.

Instead, we transform:

$$
d^2\le n
$$

into:

$$
d\le\frac nd
$$

Therefore:

```java
divisor <= n / divisor
```

avoids multiplication overflow.

## Correctness

For positive (d):

$$
d^2\le n
$$

is mathematically equivalent to:

$$
d\le\frac nd
$$

Integer division truncates the fractional part, but the comparison remains correct for positive integers in this context.

For example, let:

$$
n=10,\qquad d=3
$$

Then:

$$
3^2=9\le10
$$

and in Java:

```java
10 / 3 == 3
```

so:

```java
3 <= 3
```

is true.

For (d=4):

$$
4^2=16>10
$$

and:

```java
10 / 4 == 2
```

so:

```java
4 <= 2
```

is false.

## Complexity

$$
O(\sqrt n)
$$

Space:

$$
O(1)
$$

------

# Version 5: Skip all even numbers

Every even integer greater than (2) is composite because it can be written as:

$$
n=2k
$$

for some integer (k>1).

Therefore, after handling (2) separately, we only need to test odd divisors.

```java
public static boolean isPrimeV5(int n) {
    if (n < 2) {
        return false;
    }

    if (n == 2) {
        return true;
    }

    if (n % 2 == 0) {
        return false;
    }

    for (int divisor = 3; divisor <= n / divisor; divisor += 2) {
        if (n % divisor == 0) {
            return false;
        }
    }

    return true;
}
```

The tested divisors are now:

$$
3,5,7,9,11,\ldots
$$

instead of:

$$
2,3,4,5,6,7,\ldots
$$

## Why is skipping even divisors correct?

Any even divisor has the form:

$$
d=2k
$$

If an even divisor divides (n), then (2) also divides (n). But the method already checks:

```java
if (n % 2 == 0)
```

Therefore, after this check fails, no even number can divide (n).

## Complexity

It still has the same asymptotic complexity:

$$
O(\sqrt n)
$$

However, it performs approximately half as many divisibility tests as Version 4.

------

# Version 6: Use the (6k\pm1) optimization

This is generally the best simple implementation for checking one `int`.

```java
public static boolean isPrime(int n) {
    if (n <= 1) {
        return false;
    }

    if (n <= 3) {
        return true;
    }

    if (n % 2 == 0 || n % 3 == 0) {
        return false;
    }

    for (int divisor = 5; divisor <= n / divisor; divisor += 6) {
        if (n % divisor == 0 || n % (divisor + 2) == 0) {
            return false;
        }
    }

    return true;
}
```

The tested numbers are:

$$
5,7,\quad11,13,\quad17,19,\quad23,25,\ldots
$$

These correspond to:

$$
6k-1
$$

and:

$$
6k+1
$$

------

## Why are primes greater than 3 of the form (6k\pm1)?

Every integer can be represented in exactly one of these six forms:

$$
6k
$$

$$
6k+1
$$

$$
6k+2
$$

$$
6k+3
$$

$$
6k+4
$$

$$
6k+5
$$

Now examine each form.

### Form 1: (6k)

$$
6k=2(3k)
$$

It is divisible by (2) and (3), so it is not prime when greater than (3).

### Form 2: (6k+2)

$$
6k+2=2(3k+1)
$$

It is even, so it is not prime.

### Form 3: (6k+3)

$$
6k+3=3(2k+1)
$$

It is divisible by (3), so it is not prime.

### Form 4: (6k+4)

$$
6k+4=2(3k+2)
$$

It is even, so it is not prime.

The only remaining possibilities are:

$$
6k+1
$$

and:

$$
6k+5=6(k+1)-1
$$

Thus, every prime greater than (3) must have the form:

$$
6k-1
$$

or:

$$
6k+1
$$

------

## Important: (6k\pm1) does not guarantee primality

The theorem says:

> Every prime greater than (3) is of the form (6k\pm1).

It does not say:

> Every number of the form (6k\pm1) is prime.

For example:

$$
25=6\times4+1
$$

but:

$$
25=5\times5
$$

Similarly:

$$
35=6\times6-1
$$

but:

$$
35=5\times7
$$

Therefore, we must still test these candidates as possible divisors.

------

## Understanding the loop

```java
for (int divisor = 5; divisor <= n / divisor; divisor += 6)
```

For each loop:

```java
divisor
```

represents (6k-1), while:

```java
divisor + 2
```

represents (6k+1).

The sequence is:

| `divisor` | `divisor + 2` |
| --------- | ------------- |
| 5         | 7             |
| 11        | 13            |
| 17        | 19            |
| 23        | 25            |
| 29        | 31            |

After checking a pair, adding (6) moves to the next pair.



## Question

## 3. How does $6(k+1)-1$ become $6k-1$?

Start with

$$
6k+5.
$$

We want to express it in the form
$$
\text{a multiple of }6\text{ minus }1.
$$

Since

$$
5=6-1,
$$

we can write

$$
6k+5
=6k+(6-1).
$$

Rearranging gives

$$
6k+5
=6k+6-1.
$$

Factoring out $6$, we obtain

$$
6k+5
=6(k+1)-1.
$$

Therefore,

$$
\boxed{6k+5=6(k+1)-1}.
$$

However, $6(k+1)-1$ does **not literally become** $6k-1$ while $k$ keeps the same value. Instead, we introduce a new integer variable.

Let

$$
m=k+1.
$$

Then

$$
6(k+1)-1=6m-1.
$$

Because $m$ is simply another integer variable, we may rename $m$ as $k$. Thus, the general form can be written as

$$
6k-1.
$$

This process is called a **change of variable** or **variable substitution**.

### Example

Suppose the original value is

$$
k=3.
$$

Then

$$
\begin{aligned}
6k+5
&=6(3)+5 \\
&=23.
\end{aligned}
$$

Using the transformed expression,

$$
\begin{aligned}
6(k+1)-1
&=6(3+1)-1 \\
&=6(4)-1 \\
&=23.
\end{aligned}
$$

Notice that the multiplier in the second expression is $4$, not the original value $3$. In other words,

$$
m=k+1=4.
$$

Therefore, the precise mathematical idea is

$$
\left\{6k+5\mid k\in\mathbb{Z}\right\}
=
\left\{6m-1\mid m\in\mathbb{Z}\right\}.
$$

Both expressions describe the same set of integers.

Equivalently, because the variable name is arbitrary, we can also write

$$
\left\{6k+5\mid k\in\mathbb{Z}\right\}
=
\left\{6k-1\mid k\in\mathbb{Z}\right\}.
$$

The two occurrences of $k$ on the left and right are **independent dummy variables**; they do not represent the same fixed integer value.

------

# Complete correctness proof of Version 6

We need to prove two things:

1. If the method returns `false`, then (n) is not prime.
2. If the method returns `true`, then (n) is prime.

## Case 1: The method returns `false`

It returns `false` in one of these situations.

### (n\le1)

By definition, primes must be greater than (1). Therefore, (n) is not prime.

### (n) is divisible by (2) or (3)

Because (2) and (3) have already been handled separately, such an (n) has a proper divisor. Therefore, it is composite.

### A loop divisor divides (n)

The loop finds some integer (d) such that:

$$
1<d<n
$$

and:

$$
d\mid n
$$

Therefore, (n) is composite.

Thus, whenever the method returns `false`, the result is correct.

## Case 2: The method returns `true`

Suppose, for contradiction, that the method returns `true`, but (n) is composite.

Because (n) is composite, it has a divisor (d) such that:

$$
2\le d\le\sqrt n
$$

The method has already shown that (d) is not (2) or (3), and (d) cannot be divisible by (2) or (3). Otherwise, (n) would also be divisible by (2) or (3).

Therefore, (d) must have the form:

$$
6k-1
$$

or:

$$
6k+1
$$

The loop checks every positive number of these forms up to (\sqrt n). Therefore, it would find (d) and return `false`.

This contradicts our assumption that the method returned `true`.

Therefore, (n) must be prime.

------

# Time complexity of Version 6

The loop tests candidates only up to:

$$
\sqrt n
$$

There are approximately (\sqrt n/6) loop iterations, with two divisibility checks per iteration.

Therefore:

$$
T(n)=O(\sqrt n)
$$

Space complexity:

$$
S(n)=O(1)
$$

Skipping multiples of (2) and (3) improves the constant factor, but does not change the asymptotic complexity.

------

# Recommended implementation for Java `int`

```java
public final class PrimeNumbers {

    private PrimeNumbers() {
        // Prevent instantiation.
    }

    public static boolean isPrime(int n) {
        if (n <= 1) {
            return false;
        }

        if (n <= 3) {
            return true;
        }

        if (n % 2 == 0 || n % 3 == 0) {
            return false;
        }

        for (int divisor = 5; divisor <= n / divisor; divisor += 6) {
            if (n % divisor == 0 || n % (divisor + 2) == 0) {
                return false;
            }
        }

        return true;
    }
}
```

Example:

```java
public class Main {
    public static void main(String$$$$ args) {
        System.out.println(PrimeNumbers.isPrime(-7)); // false
        System.out.println(PrimeNumbers.isPrime(0));  // false
        System.out.println(PrimeNumbers.isPrime(1));  // false
        System.out.println(PrimeNumbers.isPrime(2));  // true
        System.out.println(PrimeNumbers.isPrime(3));  // true
        System.out.println(PrimeNumbers.isPrime(4));  // false
        System.out.println(PrimeNumbers.isPrime(17)); // true
        System.out.println(PrimeNumbers.isPrime(25)); // false
    }
}
```

------

# For a Java `long`

The same algorithm can be used:

```java
public static boolean isPrime(long n) {
    if (n <= 1) {
        return false;
    }

    if (n <= 3) {
        return true;
    }

    if (n % 2 == 0 || n % 3 == 0) {
        return false;
    }

    for (long divisor = 5; divisor <= n / divisor; divisor += 6) {
        if (n % divisor == 0 || n % (divisor + 2) == 0) {
            return false;
        }
    }

    return true;
}
```

This is mathematically correct for all positive `long` values. However, it may be too slow for very large values.

For example, if:

$$
n\approx 10^{18}
$$

then:

$$
\sqrt n\approx10^9
$$

Testing hundreds of millions of possible divisors is expensive. For arbitrary 64-bit integers, a deterministic Miller–Rabin test is normally a better practical choice.

------

# When checking many numbers: Sieve of Eratosthenes

If you need to determine primality for every number from (0) to some limit, repeatedly calling `isPrime` is not optimal.

Use a sieve:

```java
public static boolean$$$$ createPrimeTable(int limit) {
    if (limit < 0) {
        throw new IllegalArgumentException("Limit cannot be negative.");
    }

    boolean$$$$ isPrime = new boolean$$limit + 1$$;

    if (limit < 2) {
        return isPrime;
    }

    java.util.Arrays.fill(isPrime, true);
    isPrime$$0$$ = false;
    isPrime$$1$$ = false;

    for (int prime = 2; prime <= limit / prime; prime++) {
        if (!isPrime$$prime$$) {
            continue;
        }

        for (int multiple = prime * prime;
             multiple <= limit;
             multiple += prime) {
            isPrime$$multiple$$ = false;
        }
    }

    return isPrime;
}
```

## Why start from `prime * prime`?

When processing a number (p), the smaller multiples:

$$
2p,3p,\ldots,(p-1)p
$$

have already been marked by smaller factors.

For example, when processing (p=5):

$$
5\times2=10
$$

was marked while processing (2).

$$
5\times3=15
$$

was marked while processing (3).

$$
5\times4=20
$$

was marked while processing (2).

Therefore, the first potentially unmarked multiple is:

$$
p^2
$$

The sieve has time complexity:

$$
O(n\log\log n)
$$

and space complexity:

$$
O(n)
$$

------

# Final choice

| Situation                                | Recommended algorithm        |
| ---------------------------------------- | ---------------------------- |
| Check one small or medium `int`          | Trial division with (6k\pm1) |
| Check one moderate `long`                | Trial division with (6k\pm1) |
| Check a very large 64-bit number         | Deterministic Miller–Rabin   |
| Check many numbers up to a limit         | Sieve of Eratosthenes        |
| Generate primes in a very large interval | Segmented sieve              |

For most beginner and interview problems involving one Java `int`, Version 6 is the best balance of correctness, readability, speed, and overflow safety.

Supple