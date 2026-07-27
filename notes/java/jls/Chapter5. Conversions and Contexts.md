# Chapter 5. Conversions and Contexts

>  Q: `byte` and `short` operands are promoted to `int` — correct

A more precise wording for this would be:

> In most numeric arithmetic expressions, operands of type `byte` and `short` are promoted to `int`.

> Q: All types are promoted to `double` — incorrect

The promotion rules use this order:

1. If one operand is `double`, promote to `double`.
2. Otherwise, if one operand is `float`, promote to `float`.
3. Otherwise, if one operand is `long`, promote to `long`.
4. Otherwise, promote to `int`.

So types are **not always promoted to `double`**. 

Examples:

```java
int + int       // result: int
int + long      // result: long
int + float     // result: float
float + double  // result: double
```

>Q: `byte` can automatically convert directly to `char` — incorrect in the general case

The list of widening conversions from `byte` includes:

- `byte` → `short`
- `byte` → `int`
- `byte` → `long`
- `byte` → `float`
- `byte` → `double`

It does **not** include `byte` → `char`. 

The exact `byte`-to-`char` rule is here:

 <a href="https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.1.4" style="text-decoration: none;">§5.1.4 Widening and Narrowing Primitive Conversion</a>

The JLS explains that `byte` → `char` first converts the `byte` to `int`, and then narrows the `int` to `char`. It is not an ordinary widening conversion. 

Therefore, this normally fails:

There is a special exception for certain compile-time constant expressions whose values fit in `char`, but that does not make general `byte` variables automatically convertible to `char`. 

> An operation involving `float` and `double` produces `double` — correct

The rule says:

> If any expression has type `double`, the promoted type is `double`, and the other numeric expressions are widened to `double`.

Therefore:

```
float first = 1.0F;
double second = 2.0;

double result = first + second;
```

Before the addition:

```
float + double
    ↓
double + double
    ↓
double
```

Several operands

```java
byte byteNumber = 1;
int intNumber = 2;
float floatNumber = 3.0F;
double doubleNumber = 4.0;

double result = byteNumber + intNumber + floatNumber + doubleNumber;
```

The promotion happens step by step:

**`byte + int → int`**

**`int + float → float`**

**`float + double → double`**

Therefore, the final result has type:

The central rule is:

> When a numeric operation contains a `double` operand, the other numeric operand is widened to `double`, and the result of that operation is `double`.






## Narrowing Primitive Conversion and Two's-Complement Representation

The following code does **not** compile because Java does not implicitly narrow an `int` variable to `byte`:

```java
int a = 135;
byte b = a; // compile-time error
```

An explicit cast is required:

```java
int a = 135;
byte b = (byte) a;
System.out.println(b); // -121
```

### 1. Narrowing from `int` to `byte`

An `int` uses 32 bits, while a `byte` uses 8 bits. During the narrowing conversion, only the lowest 8 bits are retained.

The 32-bit representation of `135` is:

```text
00000000 00000000 00000000 10000111
```

After narrowing to `byte`, the remaining bits are:

```text
10000111
```

Java's `byte` is an 8-bit **signed two's-complement integer**, so this bit pattern must be interpreted as a signed value.

### 2. Unsigned binary weights

For an unsigned 8-bit integer, the bit weights are:

| Bit position | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Weight | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

Therefore, if `10000111` were unsigned, its value would be:

$$128 + 4 + 2 + 1 = 135$$

### 3. Signed two's-complement weights

For an 8-bit signed two's-complement integer, the highest bit has a negative weight:

| Bit position | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Weight | **-128** | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

The `1` bits in `10000111` are at positions 7, 2, 1, and 0. Therefore:

$$-128 + 4 + 2 + 1 = -121$$

That is why `(byte) 135` is `-121`.

For an $n$-bit two's-complement number, the general value formula is:

$$
-b_{n-1}2^{n-1} + \sum_{i=0}^{n-2} b_i2^i
$$

For 8 bits, this becomes:

$$
-b_7 \times 128 + b_6 \times 64 + b_5 \times 32 + \cdots + b_0 \times 1
$$

Each $b_i$ is either `0` or `1`.

### 4. Reading a negative value by inverting the bits and adding 1

Another way to interpret `10000111` is:

1. The highest bit is `1`, so the number is negative.
2. Invert every bit:

```text
10000111
↓
01111000
```

3. Add `1`:

```text
01111000 + 1 = 01111001
```

4. Convert `01111001` to decimal:

$$64 + 32 + 16 + 8 + 1 = 121$$

Therefore, the original value is:

$$-121$$

### 5. Creating a negative number in two's complement

To represent `-5` using 8 bits:

1. Write positive `5`:

```text
00000101
```

2. Invert every bit to obtain the **one's complement**:

```text
11111010
```

3. Add `1` to obtain the **two's complement**:

```text
11111010 + 1 = 11111011
```

Therefore:

```text
 5 = 00000101
-5 = 11111011
```

The signed-weight method confirms it:

$$-128 + 64 + 32 + 16 + 8 + 2 + 1 = -5$$

### 6. One's complement and two's complement

**One's complement** means inverting every bit:

```text
0 → 1
1 → 0
```

**Two's complement** means:

1. Invert every bit.
2. Add `1`.

Two's complement is commonly used because:

- zero has only one representation;
- the same binary addition circuit works for positive and negative numbers;
- subtraction can be performed as addition of a negative number.

For example:

$$5 - 3 = 5 + (-3)$$

```text
  00000101   // 5
+ 11111101   // -3
-----------
1 00000010
```

Only 8 bits are retained, so the carry on the left is discarded:

```text
00000010 = 2
```

### 7. Why the `byte` range is `-128` to `127`

Eight bits provide:

$$2^8 = 256$$

possible bit patterns.

In two's-complement representation:

- `00000000` to `01111111` represent `0` to `127`;
- `10000000` to `11111111` represent `-128` to `-1`.

Important boundary values are:

| Binary | Decimal |
|---|---:|
| `01111111` | 127 |
| `10000000` | -128 |
| `10000111` | -121 |
| `11111111` | -1 |

This is why there is no positive `128` in the Java `byte` type.

### 8. The modulo-256 interpretation

An 8-bit value has 256 possible patterns, so narrowing can also be understood using arithmetic modulo 256.

For `135`:

$$135 - 256 = -121$$

Both `135` as an unsigned value and `-121` as a signed value correspond to the same 8-bit pattern:

```text
10000111
```

The bits do not change; only their interpretation changes.

### 9. Overflow and Java arithmetic promotion

In a pure 8-bit two's-complement calculation:

```text
01111111 + 00000001 = 10000000
```

This means:

$$127 + 1 \rightarrow -128$$

However, Java normally promotes `byte` and `short` operands to `int` before arithmetic. Therefore:

```java
byte x = 127;
byte y = (byte) (x + 1);
System.out.println(y); // -128
```

The cast is required because `x + 1` has type `int`.

The central rules are:

> During an `int`-to-`byte` narrowing conversion, Java retains the lowest 8 bits.

> In an 8-bit two's-complement signed number, the highest bit has weight `-128`, while the remaining bits have their ordinary positive weights.
